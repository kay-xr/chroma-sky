# ChromaSky

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2FuhKayla%2Fchroma-sky)

ChromaSky is a small app for collecting a Bluesky user's photos and displaying them on a single page. This is good for photo-focused accounts.

The app will find all photos with specified tags, and display them. This allows you to use custom keywords to filter photos for your site.

The recommendation is to deploy this via Vercel or equivalent and display it on your handle URL.

Below are some more detailed instructions for setup and use.

### Configuration

Configuration is handled via the `config.ts` file located at `src/lib/config.ts`

You can specify your handle in `username`
Specify your specific tag in the `keywords` array, you can also leave this blank to include all photos.

At the moment ChromaSky will show the most recent photos, up to 100, with pagination planned for the future.

### To Deploy

To deploy this repo, start by forking it by clicking the button in the upper right.\
Next, modify the config file, as described above, to customize the app for your account.\
Finally, deploy the site via a service like Vercel. You can do this by connecting your clicking the button at the top of the page. \
You can also link your handle to be the primary domain for your site.\