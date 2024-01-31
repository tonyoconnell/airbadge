## Stripe + Auth.js + SvelteKit

The easiest way to create a SaaS site.

This is a [Stripe](https://stripe.com) integration for [Auth.js](https://authjs.dev).

It's a quick way to create SaaS apps without writing any payment code.

## Features

- **Integrated checkout**: Stripe Checkout is built into the signup flow.
- **Billing portal**: Changing plans or canceling plans is built in.
- **Webhook handling**: All Stripe webhooks are handled for you.
- **Free plans**: Checkout can be skipped for free plans or trials.
- **Routing guards**: Routes can be restricted based on membership status.
- **Component guards**: Conditionally display components based on membership status.
- **Open source**: https://github.com/joshnuss/auth-stripe-sveltekit
- **BSL Licence**: Free for first 100 customers. Then $150/year for unlimited users.

## Component guards

Conditionally display components based on the user's subscription status.

Two component wrappers are provided:

- `<NonMember/>`: Display content when user doesn't have a subscription.
- `<Member/>`: Display content when user has a subscription. Can also filter by plan or payment state.

### Examples

```html
<script>
  import { Member, NonMember } from 'auth-stripe-sveltekit'
</script>

<!-- show to all members -->
<Member>
  <p>Welcome back member!</p>
</Member>

<!-- show to unpaid members -->
<Member unpaid>
  <p>Whoops, we couldn't collect a payment.</p>

  <a href="/billing/portal">Upgrade</a>
</Member>

<!-- show to members with canceled subscriptions -->
<Member canceled>
  <p>Your account has been canceled</p>

  <a href="/billing/checkout">Sign up</a>
</Member>

<!-- show to members on the "pro" plan -->
<Member plan="pro">
  You're on the Pro plan!!
</Member>

<!-- show to members on the "pro" or "enterprise" plan -->
<Member plans={["pro", "enterprise"]}>
  You're a real player!!
</Member>

<!-- show to non-members -->
<NonMember>
  <a href="/billing/checkout">Sign up</a>
</NonMember>
```

## Routing guards

Guards are helper functions that can restrict access to routes based on the state of the subscription:

```javascript
// in +page.server.js
import { nonMember, member } from 'auth-stripe-sveltekit'

// route is for members only (including canceled, or late on payment)
export const load = member(callback)

// route is for fully paid members only
export const load = member.active(callback)

// route is for past due members only
export const load = member.past_due(callback)

// route is for unpaid members only
export const load = member.unpaid(callback)

// route is for members that have canceled their subscription
export const load = member.canceled(callback)

// route is for members on the "pro" plan
export const load = member.plan('pro', callback)

// route is for members on the "pro" or "enterprise" plans
export const load = member.plans(['pro', 'enterprise'], callback)

// route is for non-members only
export const load = nonMember(callback)
```

## Billing Routes

This package provides a `/billing` route, similar to how Auth.js provides a `/auth` route.

The following routes are provided:

- `/billing/checkout?plan=<plan>`: Begin checkout flow for a specific plan. If no `plan` is specified, the default plan is used.
- `/billing/portal`: Opens the billing portal for the current signed-in user.
- `/billing/cancel`: Cancels the current user's subscription.
- `/billing/webhooks`: Handles all Stripe webhooks for you.
- `/billing/plans`: List plans in json format.

## Setup

Install [auth-stripe-sveltekit](https://npmjs.com/package/auth-stripe-sveltekit):

```sh
pnpm i -D auth-stripe-sveltekit
```

Setup a database provider for Auth.js. For example, follow instructions for Prisma:

https://authjs.dev/reference/adapter/prisma

Add environment variables to `.env`:

```sh
PUBLIC_STRIPE_KEY=pk_...
SECRET_STRIPE_KEY=sk_...
DOMAIN=http://localhost:5173
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/auth_stripe_sveltekit_dev?schema=public"
```

Configure authentication and billing options in `src/hooks.server.js`:

```javascript
import { StripeSvelteKitAuth } from 'auth-stripe-sveltekit'

// use any OAuth provider (or multiple)
import GitHub from '@auth/core/providers/github'

// import prisma client for Auth.js's database adapter
import { PrismaClient } from '@prisma/client'

// init database client
const db = new PrismaClient()

// add Auth.js + Stripe handler
// API is similar to Auth.js
export const handle = StripeSvelteKitAuth({
  // configure database adapter
  adapter: PrismaAdapter(db),

  // configure OAuth providers
	providers: [
		GitHub({
			clientId: env.GITHUB_ID,
			clientSecret: env.GITHUB_SECRET
		})
	],

  // configure list of plans.
  // free and trial plans are supported
  plans: [
    { id: 'basic', name: 'Basic', free: true, default: true },
    { id: 'pro', name: 'Pro', trial: true }
    { id: 'enterprise', name: 'Enterprise', trial: true }
  ]
})
```

## License

BSL - Business Software License.

Free to use for first 100 users, then $150 USD/year for unlimited usage.
