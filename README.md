# Next.js Notes

## Intro - Workspace Setup

```bash
pnpm init
```

```bash
echo "packages:\n  - \"packages/*\"" >> pnpm-workspace.yaml
```

## Chapter 1 - Dashboard

`npx create-next-app@latest packages/01-nextjs-dashboard --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example" --use-pnpm`

`pnpm create next-app@latest packages/01-nextjs-dashboard --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example"`

**Folder Structure:**

- /app: Contains all the routes, components, and logic for your application, this is where you'll be mostly working from.
- /app/lib: Contains functions used in your application, such as reusable utility functions and data fetching functions.
- /app/ui: Contains all the UI components for your application, such as cards, tables, and forms. To save time, we've pre-styled these components for you.
- /public: Contains all the static assets for your application, such as images.

Config Files: You'll also notice config files such as `next.config.js` at the root of your application. Most of these files are created and pre-configured when you start a new project using `create-next-app`. You will not need to modify them in this course.

**TypeScript:**

We're manually declaring the data types, but for better type-safety, we recommend `Prisma` or `Drizzle`, which automatically generates types based on your database schema.
Next.js detects if your project uses TypeScript and automatically installs the necessary packages and configuration. Next.js also comes with a TypeScript plugin
for your code editor, to help with auto-completion and type-safety.

**Run the app:**

`pnpm -F 01-nextjs-dashboard dev`

## Chapter 2 - Styling

- How to add a global CSS file to your application.
- Two different ways of styling: Tailwind and CSS modules.
- How to conditionally add class names with the clsx utility package.

**Global CSS:**

Defined in `/app/ui/global.css` and imported in `/app/layout.tsx`
You can import `global.css` in any component however, it's commone to add it to your top-level component i.e. the root layout.

**Tailwind:**

Tailwind is a CSS framework that speeds up the development process by allowing you to quickly write utility classes directly in your TSX markup.

Although the CSS styles are shared globally, each class is singularly applied to each element. This means if you add or delete an element, you don't have to worry about maintaining separate stylesheets, style collisions, or the size of your CSS bundle growing as your application scales.

**CSS Modules**

If you prefer writing traditional CSS rules or keeping your styles separate from your JSX - CSS Modules are a great alternative.
CSS Modules allow you to scope CSS to a component by automatically creating unique class names, so you don't have to worry about style collisions as well.

**clsx**

clsx is a library that lets you toggle class names conditionally.

```tsx
import clsx from 'clsx';
 
export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
    // ...
  );
}
```

**Other styling solutions**

In addition to the approaches we've discussed, you can also style your Next.js application with:
- Sass which allows you to import .css and .scss files.
- CSS-in-JS libraries such as styled-jsx, styled-components, and emotion.

## Chapter 3 - Fonts and Images

[Cumulative Layout Shift](https://vercel.com/blog/how-core-web-vitals-affect-seo) is a metric used by Google to evaluate the performance and user experience of a website. With fonts, layout shift happens when the browser initially renders text in a fallback or system font and then swaps it out for a custom font once it has loaded. This swap can cause the text size, spacing, or layout to change, shifting elements around it.

Next.js automatically optimizes fonts in the application when you use the next/font module. It downloads font files at build time and hosts them with your other static assets. This means when a user visits your application, there are no additional network requests for fonts which would impact performance.

/app/ui/fonts.ts

```tsx
import { Inter } from 'next/font/google';
 
export const inter = Inter({ subsets: ['latin'] });
```

/app/layout.tsx

```tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

**Why optimize images?**

Next.js can serve static assets, like images, under the top-level /public folder. Files inside /public can be referenced in your application.

With regular HTML, you would add an image as follows:

<img
  src="/hero.png"
  alt="Screenshots of the dashboard project showing desktop version"
/>

However, this means you have to manually:
- Ensure your image is responsive on different screen sizes.
- Specify image sizes for different devices.
- Prevent layout shift as the images load.
- Lazy load images that are outside the user's viewport.

Image Optimization is a large topic in web development that could be considered a specialization in itself. Instead of manually implementing these optimizations, you can use the next/image component to automatically optimize your images.

/app/page.tsx
```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
import Image from 'next/image';
 
export default function Page() {
  return (
    // ...
    <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
      {/* Add Hero Images Here */}
      <Image
        src="/hero-desktop.png"
        width={1000}
        height={760}
        className="hidden md:block"
        alt="Screenshots of the dashboard project showing desktop version"
      />
    </div>
    //...
  );
}
```

Here, you're setting the `width` to `1000` and `height` to `760` pixels. It's good practice to set the `width` and `height` of your images to avoid layout shift, these should be an aspect ratio identical to the source image.

**Remeber** Images without dimensions and web fonts are common causes of layout shift.

**Recommended reading**
- [Font Optimization Docs](https://nextjs.org/docs/app/building-your-application/optimizing/images)
- [Image Optimization Docs](https://nextjs.org/docs/app/building-your-application/optimizing/fonts)
- [Improving Web Performance with Images (MDN)](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Performance/Multimedia)
- [Web Fonts (MDN)](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Text_styling/Web_fonts)
- [How Core Web Vitals Affect SEO](https://vercel.com/blog/how-core-web-vitals-affect-seo)

## Chapter 4 - Layouts and Pages

**Nested routing**

Next.js uses file-system routing where folders are used to create nested routes. Each folder represents a route segment that maps to a URL segment.

You can create separate UIs for each route using layout.tsx and page.tsx files.

`page.tsx` is a special Next.js file that exports a React component, and it's required for the route to be accessible. In your application, you already have a `page.tsx` file: `/app/page.tsx` - this is the home page associated with the route `/`.

To create a nested route, you can nest folders inside each other and add `page.tsx` files inside them

`/app/dashboard/page.tsx` is associated with the `/dashboard` path
```tsx
export default function Page() {
  return <p>Dashboard Page</p>;
}
```

This is how you can create different pages in Next.js: create a new route segment using a folder, and add a page file inside it.

By having a special name for `page` files, Next.js allows you to colocate UI components, test files, and other related code with your routes. Only the content inside the `page` file will be publicly accessible. For example, the `/ui` and `/lib` folders are colocated inside the `/app` folder along with your routes.

**Creating the dashboard layout**

Dashboards have some sort of navigation that is shared across multiple pages. In Next.js, you can use a special `layout.tsx` file to create UI that is shared between multiple pages. Let's create a layout for the dashboard pages!

Inside the `/dashboard` folder, add a new file called `layout.tsx` and paste the following code:

`/app/dashboard/layout.tsx`
```tsx
import SideNav from '@/app/ui/dashboard/sidenav';
 
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
      <div className="w-full flex-none md:w-64">
        <SideNav />
      </div>

      <div className="flex-grow p-6 md:overflow-y-auto md:p-12">
        {/* This child can either be a page or another layout */}
        {children}
      </div>
    </div>
  );
}
```

One benefit of using `layouts` in Next.js is that on navigation, only the page components update while the layout won't re-render. This is called partial rendering:

**Root Layout**

`/app/layout.tsx` is called a root layout and is required. Any UI you add to the root layout will be shared across all pages in your application. You can use the root layout to modify your <html> and <body> tags, and add metadata

## Chapter 5 - Navigation

When using a traditionally <a> HTML element to link pages the will perform a full page refresh on each page navigation!

**The <Link> component**

Use the <Link /> Component to link between pages in your application allowing client-side navigation with JavaScript.

**Automatic code-splitting and prefetching**

Next.js automatically code splits your application by route segments. This is different from a traditional React SPA, where the browser loads all your application code on initial load.

Splitting code by routes means that pages become isolated. If a certain page throws an error, the rest of the application will still work.

Furthermore, in production, whenever <Link> components appear in the browser's viewport, Next.js automatically prefetches the code for the linked route in the background. By the time the user clicks the link, the code for the destination page will already be loaded in the background, and this is what makes the page transition near-instant!

## Chapter 6 - Setting Up Your Database

Setup a PostgreSQL database from one of Vercel's marketplace integrations. You can skip this step if you're using your own database provider.

**Seed your database**

Uncomment `/app/seed/route.ts`. This creates a server-side endpoint that you can access in the browser to start populating your database.

Ensure your local development server is running with pnpm run dev and navigate to `localhost:3000/seed`
in your browser. When finished, you will see a message "Database seeded successfully" in the browser. Once completed, you can delete this file.!

If you run into any issues while seeding your database and want to run the script again, you can drop any existing tables by running `DROP TABLE tablename` in your database query interface. See the executing queries section below for more details. But be careful, this command will delete the tables and all their data. It's ok to do this with your example app since you're working with placeholder data, but you shouldn't run this command in a production app.

**Executing queries**

Let's execute a query to make sure everything is working as expected. We'll use another Router Handler, `query/route.ts`, to query the database. Inside this file, you'll find a `listInvoices()` function that has the following SQL query.

```sql
SELECT invoices.amount, customers.name
FROM invoices
JOIN customers ON invoices.customer_id = customers.id
WHERE invoices.amount = 666;
```

Uncomment the file and navigate to `localhost:3000/query` in your browser. You should see that an invoice amount and name is returned.
