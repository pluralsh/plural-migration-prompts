Source system: Argo CD
Output mode: yaml
Current state:
- `checkout-dev` deploys the `services/checkout/overlays/dev` folder from `ssh://git@github.com/acme/platform-infra.git` to cluster `dev-us-east-1` in namespace `checkout`.
- `checkout-prod` deploys the `services/checkout/overlays/prod` folder from the same repo to cluster `prod-us-east-1` in namespace `checkout`.
- The platform team wants production changes to happen only after the dev service is healthy and a human approves promotion.
- This is not fleet fan-out. It is a dev to prod promotion workflow.
