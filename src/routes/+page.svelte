<script lang="ts">
	import { config } from '$lib/config';
	import Masonry from 'svelte-bricks';
	import { onMount } from 'svelte';
	import * as Dialog from "$lib/components/ui/dialog";
	import { Button } from "$lib/components/ui/button";
	import Waves from '$lib/components/Waves.svelte';
	import Footer from '$lib/components/Footer.svelte';
	import * as Card from "$lib/components/ui/card";

	let authorDisplayName: string;
	let authorBio: string;
	let posts: any[] = []; 			// Store full post objects
	let images: string[] = []; 	// Store image URLs for Masonry component

	$: items = [...images]; 		// Map images to the reactive array used by Masonry

	let minColWidth = 600; 			// Minimum column width
	let gap = 0; 								// Gap between items
	let maxColWidth = 1200; 		// Default fallback column width

	onMount(async () => {
		// Get author displayname
		const author = await fetch(
			`https://public.api.bsky.app/xrpc/app.bsky.actor.getProfile?actor=${config.username}`
		);
		const authorResponse = await author.json();

		authorDisplayName = authorResponse.displayName;
		authorBio = authorResponse.description;

		// Get posts
		const response = await fetch(
			`https://public.api.bsky.app/xrpc/app.bsky.feed.getAuthorFeed?actor=${config.username}&filter=posts_with_media`
		);
		const data = await response.json();

		console.log(data);

		// Filter posts with images
		let filteredPosts = data.feed.filter(post => post.post.embed?.images?.length > 0);

		// Conditionally filter posts by keywords
		if (config.keywords.length > 0) {
			filteredPosts = filteredPosts.filter(post => {
				const text = post.post.record.text || '';
				return config.keywords.some(keyword => text.includes(keyword));
			});
		}

		// Store filtered posts and their images
		posts = filteredPosts.map(post => ({
			key: post.post.cid,
			...post
		}));
		images = posts.flatMap(post => post.post.embed.images.map(img => img.fullsize)); // Extract image URLs
	});
</script>

<Waves />

<div class="grid grid-cols-2">
	<div class="flex flex-col w-full justify-start items-start gap-2 p-20 pb-0">
		<div class="card-test ">
			<Card.Root>
				<Card.Header>
					<h1 class="text-2xl font-bold">{authorDisplayName}'s photos</h1>
				</Card.Header>
				<Card.Content>
					<p class="text-sm text-gray-600">{authorBio}</p>
				</Card.Content>
			</Card.Root>
		</div>
	</div>
	<div class="flex flex-col w-full justify-end items-end gap-2 p-20 pb-0">
		<div class="card-test ">
			<Button href="https://bsky.app/profile/{config.username}">Follow on Bluesky</Button>
		</div>
	</div>
</div>

{#if items.length > 0}
	<Masonry {items} {minColWidth} {maxColWidth} {gap} let:item class="p-16">
		{#each posts as post (post.key)}
			{#if post.post.embed.images.some(img => img.fullsize === item)}
				<Dialog.Root>
					<Dialog.Trigger class="w-full">
						<div class="image-container p-4 w-full min-w-full">
							<img src="{item}" alt="{post.post.record.text}" class="image w-full h-auto" loading="lazy" />
						</div>
					</Dialog.Trigger>
					<Dialog.Content class="sm:max-w-[425px] md:max-w-[640px] lg:max-w-[800px] xl:max-w-[800px]">
						<img src="{item}" alt="{post.post.record.text}" class="w-full h-auto" loading="lazy" />
						<p class="mt-2 text-lg">{post.post.record.text}</p>
						<Dialog.Footer>

						</Dialog.Footer>
					</Dialog.Content>
				</Dialog.Root>
			{/if}
		{/each}
	</Masonry>
{:else}
	<p class="text-gray-500">Loading photos...</p>
{/if}

<Footer name="{authorDisplayName}" link="https://bsky.app/profile/{config.username}"/>

<style>
    @media (min-width: 1024px) {
        :global(.masonry) {
            grid-template-columns: repeat(4, 1fr);
        }
    }

    @media (min-width: 768px) and (max-width: 1023px) {
        :global(.masonry) {
            grid-template-columns: repeat(3, 1fr);
        }
    }

    @media (min-width: 640px) and (max-width: 767px) {
        :global(.masonry) {
            grid-template-columns: repeat(2, 1fr);
        }
    }

    @media (max-width: 639px) {
        :global(.masonry) {
            grid-template-columns: repeat(1, 1fr);
        }
    }

    .image-container {
        overflow: hidden; /* Ensures images don't overflow during scale */
        width: 100%;
        min-width: 100%;
    }

    .image {
        transition: transform 0.3s ease, box-shadow 0.3s ease; /* Smooth scaling */
        border-radius: var(--radius);
        width: 100%;
				min-width: 100%;
    }

    .image:hover {
        transform: scale(1.05); /* Slightly enlarge the image */
        /*box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3); !* Add a shadow for depth *!*/
    }

		.card-test {
        transition: transform 0.3s ease, box-shadow 0.3s ease; /* Smooth scaling */
		}

		.card-test:hover {
				transform: scale(1.05);
		}
</style>
