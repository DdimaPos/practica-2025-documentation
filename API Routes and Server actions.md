# API Routes and Server Actions

As discussed there are multiple ways to handle data in Next js.
From what we now for now those are:

- Server actions
- API routes defined in `app/api` directory

Here I want to leave the breakdown from gemini to properly understand when you should use server actions and when to use api endpoints.
From my perspective it is valid.

> [!note] A Simple Rule to decide
> Is the "customer" of this logic your Next.js components?
>
> - Use a Server Action.
>   Is the "customer" an external service, a mobile app, or a different website?
> - Use an API Route.

### Server Actions

Server Actions are the **default, modern choice for mutations and server communication _within_ your Next.js application**. They are tightly integrated with the React Server Components paradigm.

#### 1. Form Submissions and Data Mutations (CRUD)

This is the primary and most powerful use case. Anytime a user needs to create, update, or delete data.

- **Examples:**
  - Submitting a contact form.
  - Posting a comment on a blog.
  - Updating user profile information.
  - Deleting an item from a list.
- **Why:** Server Actions dramatically simplify the process. You get progressive enhancement for free (forms work without JS), and you can avoid all the client-side boilerplate for managing state, loading, errors, and re-fetching data thanks to built-in features like `revalidatePath`.

#### 2. Simple UI Interactions that Require Server Logic

For actions triggered by buttons or other UI elements that don't belong in a full `<form>`.

- **Examples:**
  - Clicking a "like" button on a post.
  - Adding an item to a shopping cart.
  - Subscribing to a newsletter with just an email field and a button.
- **Why:** You can bind a Server Action directly to an event handler. This keeps your logic co-located and provides a seamless way to update the UI on the server, sending back the refreshed component tree.

#### 3. Securely Calling Third-Party APIs

When your application needs to fetch data from an external service, but you need to protect your API keys or secrets.

- **Examples:**
  - Fetching shipping rates from a provider's API.
  - Processing a payment with Stripe.
  - Getting data from a private CMS.
- **Why:** The Server Action runs exclusively on the server, so your secret keys are never exposed to the client's browser. It acts as a secure proxy for your components.

#### 4. Running Server-Only Operations

For tasks that have nothing to do with returning data to the client, but are triggered by user interaction.

- **Examples:**
  - Triggering a database cleanup job.
  - Sending an email notification.
  - Logging a specific user event to an analytics service.
- **Why:** It's a direct and secure hook for a component to execute a function in your server environment.

---

### API Routes (Endpoints)

API Routes are for building a traditional, **decoupled API layer**. You should use them whenever your backend needs to communicate with anything _outside_ of your Next.js React components.

#### 1. Providing a Backend for External Clients

This is the most common reason. Your Next.js app is serving as the backend for another application.

- **Examples:**
  - A mobile app (iOS/Android) that needs to fetch user data.
  - A separate, single-page application (built in Vue, Angular, or even another React app) that consumes your data.
  - Desktop applications or IoT devices.
- **Why:** These clients don't understand Server Actions. They speak the universal language of HTTP. An API Route provides a standard, stateless `GET /api/users` or `POST /api/products` endpoint they can reliably call.

#### 2. Handling Webhooks

When you need to receive data from third-party services.

- **Examples:**
  - Stripe sending a `payment_succeeded` event.
  - GitHub sending a notification when a `git push` occurs.
  - A headless CMS (like Contentful or Sanity) notifying your app that content has been updated and it should rebuild.
- **Why:** Webhooks require a stable, public URL to send a POST request to. An API Route is the perfect tool to create that publicly accessible endpoint.

#### 3. Creating a Public API for Third-Party Developers

If you are building a service that other developers will integrate into their own applications.

- **Examples:**
  - Your own version of a weather API or a stock price API.
  - An API for your SaaS product that allows customers to access their data programmatically.
- **Why:** A public API requires clear, versioned, and well-documented endpoints based on standard REST or GraphQL principles. API Routes are designed for this level of control and standardization.

#### 4. Serving Non-React Content

When the response isn't HTML/JSON intended for React components, but something else entirely.

- **Examples:**
  - Dynamically generating an RSS feed (`/api/rss.xml`).
  - Generating and serving a PDF invoice.
  - Exporting user data as a CSV file.
- **Why:** API Routes give you fine-grained control over the response, allowing you to set specific `Content-Type` headers (e.g., `application/xml`, `application/pdf`) and stream file data.

### Summary Table

| Use Case                                                | Recommended Choice   | Rationale                                                           |
| :------------------------------------------------------ | :------------------- | :------------------------------------------------------------------ |
| **User updates their profile inside your app**          | ✅ **Server Action** | Tightly integrated with the UI, simple data revalidation.           |
| **A mobile app needs to fetch that user's profile**     | 🌐 **API Route**     | The mobile app needs a standard HTTP endpoint to call.              |
| **A "Like" button on a post**                           | ✅ **Server Action** | A simple server-side mutation triggered from the UI.                |
| **Stripe sends a webhook to confirm a payment**         | 🌐 **API Route**     | An external service needs a stable URL to send data to.             |
| **A developer wants to integrate your service via API** | 🌐 **API Route**     | You need to provide a public, documented, and stable API contract.  |
| **Generating an RSS feed for your blog**                | 🌐 **API Route**     | You need to serve a specific data format (XML) with custom headers. |

## Conclusion

So, I think you got the idea whats happening with both of them. Remember that if you want to defined actions for specific feature, include it there in `features/[featName]/actions/` directory.
Of course there could be exceptions, but be rational in deciding where you place your actions
