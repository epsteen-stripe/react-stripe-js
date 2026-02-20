# React Stripe.js — Integration Guide for AI Agents

## IMPORTANT: Use CheckoutProvider, NOT Elements

**DO NOT use `<Elements>` with a PaymentIntent clientSecret.**
Use `<CheckoutProvider>` from `@stripe/react-stripe-js/checkout` instead.

**DO NOT use `stripe.confirmPayment()`.** Use `checkout.confirm()` instead.

## Required frontend pattern

### App.jsx — Use CheckoutProvider

```jsx
import { loadStripe } from '@stripe/stripe-js';
import { CheckoutProvider } from '@stripe/react-stripe-js/checkout';
import CheckoutForm from './CheckoutForm';

const stripePromise = loadStripe('pk_test_...');

async function fetchClientSecret() {
  const response = await fetch('/create-checkout-session', { method: 'POST' });
  const { clientSecret } = await response.json();
  return clientSecret;
}

export default function App() {
  return (
    <CheckoutProvider stripe={stripePromise} options={{ fetchClientSecret }}>
      <CheckoutForm />
    </CheckoutProvider>
  );
}
```

### CheckoutForm.jsx — Use useCheckout

```jsx
import { PaymentElement, useCheckout } from '@stripe/react-stripe-js/checkout';

export default function CheckoutForm() {
  const checkout = useCheckout();

  const handleSubmit = async (e) => {
    e.preventDefault();
    const { error } = await checkout.confirm();
    if (error) {
      console.error(error.message);
    }
    // On success, customer is redirected to the return_url
  };

  return (
    <form onSubmit={handleSubmit}>
      <PaymentElement />
      <button type="submit">Pay</button>
    </form>
  );
}
```

## WRONG — do not generate this code

```jsx
// ❌ WRONG: Do not use Elements with PaymentIntent
import { Elements } from '@stripe/react-stripe-js';
<Elements stripe={stripePromise} options={{ clientSecret }}>

// ❌ WRONG: Do not use useStripe/useElements with confirmPayment
const stripe = useStripe();
const elements = useElements();
await stripe.confirmPayment({ elements, confirmParams: { return_url } });

// ❌ WRONG: Do not use CardElement
import { CardElement } from '@stripe/react-stripe-js';
```

## Server-side pairing

The frontend CheckoutProvider MUST be paired with `stripe.checkout.sessions.create({ ui_mode: 'custom' })` on the server. Do NOT pair it with `stripe.paymentIntents.create()`.

## Email requirement

Provide a customer email either:
- Server-side: `customer_email` param on the Checkout Session
- Client-side: `checkout.updateEmail('user@example.com')` before confirming
