## Provider Listener Quickstart (Minimal)

Add this snippet to your app (e.g., in `index.html` or your main JS entry) to accept Agentum preview auth inside an iframe.

```html
<script>
(function () {
  const allowedOrigins = [
    'http://localhost:8080',           // Dev (Agentum frontend)
    'https://your-agentum-domain.com'  // Replace with production domain
  ];

  function isAllowedOrigin(origin) {
    return allowedOrigins.includes(origin);
  }

  window.addEventListener('message', (event) => {
    if (!isAllowedOrigin(event.origin)) return;
    const msg = event.data;
    if (!msg || msg.type !== 'authenticationData') return;

    const auth = msg.data || {};
    if (!auth.preview_token) return;

    // Store token + user for your app
    window.agentumAuth = auth;
    sessionStorage.setItem('agentum_preview_token', auth.preview_token);
    sessionStorage.setItem('agentum_user', JSON.stringify(auth.user_info || {}));

    // Helper for authenticated requests to YOUR backend
    window.agentumFetch = (endpoint, opts = {}) =>
      fetch(endpoint, {
        ...opts,
        headers: { ...(opts.headers || {}), Authorization: `Bearer ${auth.preview_token}` }
      });
  });
})();
</script>
```

Usage example (in your app code):

```js
// After the message arrives
const profile = await window.agentumFetch('/api/me').then(r => r.json());
```

Notes
- Prefer token/sessionStorage auth over cookies in iframes.
- Replace the production origin in `allowedOrigins`.

