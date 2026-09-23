# SoFi site notes (from live probe 2026-09-23)

- Login is Auth0 Universal Login. Session URL looks like
  `https://login.sofi.com/u/login?state=...` — the `state` param expires.
  For credential capture use the stable page `https://login.sofi.com/u/login`
  (scheme + host + path, no query params).
- sofi.com homepage loads with no bot wall (only a privacy notice).
- The login page itself embeds a **Cloudflare Turnstile "Verify you are human"
  checkbox** between the password field and the reset link. It does not
  auto-verify. Expect the browser task to pause here unless the user has set
  a CAPTCHA preference for this origin — offer takeover or ask for a choice.
- Form: `Email*` + `Password` (with show toggle), "Log in" button. No SSO
  buttons (Google/Apple/passkey) on this screen.
- Login page mentions no 2FA/MFA step, but SoFi commonly issues a one-time
  code after password submit — handle via the standing OTP permission
  (Gmail auto-read; SMS → ask user to paste/complete).
- No hard block page or "unusual traffic" wall observed on the probe run.
  Bot posture can change without notice — on any new block, stop and report.
