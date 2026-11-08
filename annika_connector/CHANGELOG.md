# Changelog

## 1.4.0

- The app installs and updates the Annika integration (`custom_components/annika`) itself, in
  its own `integration/` process. It never picks a version: it applies the one the operator
  pinned for this house from the dashboard, and does nothing while none is pinned or the house
  is still managed through HACS.
- Every package is checked against its sha256 and an ed25519 signature before it is unpacked.
  The public key is baked into the image (`INTEGRATION_SIGNING_PUBLIC_KEY` build arg); without
  it the installer stays off.
- The new version is unpacked beside `custom_components` and swapped in atomically, so Home
  Assistant never sees a half-written copy. Home Assistant is restarted afterwards.
- On a house with no `annika:` configuration the app writes `annika.yaml` and adds
  `annika: !include annika.yaml` to `configuration.yaml`, keeping a `.annika-backup` copy.
- Checks on start and about every 12 hours, with jitter.
- Needs read/write access to the Home Assistant config folder (`map: homeassistant_config`).

## 1.3.1

- Fix a restart loop: the certificate renewal timer was a single `setTimeout` ~60 days out,
  which overflows `2^31-1` ms and fires immediately. It is now an hourly check.

## 1.3.0

- Local access from the home network: a TLS listener on port 8124 that serves this unit's
  Let's Encrypt certificate and proxies to Home Assistant without going through the tunnel.
  Lives in its own `local-access/` app, started alongside the tunnel.
- The private key for that certificate is generated in the house and never leaves it; only a
  CSR is sent to the backend.
- Tickets are verified offline against a cached JWKS, so local access keeps working while
  the WAN is down.
- Reports the host's LAN address on start and once an hour, so its DNS record stays correct.
- New `local_access` option, on by default. With it off, nothing listens on 8124.
- Local access never affects the tunnel: whatever fails there, rathole stays up.

## 1.2.0

- The add-on is now supervised by a small Node process instead of `run.sh`. Option parsing
  and validation, gateway registration with retries, the saved assignment and the rathole
  client config all moved out of bash and are covered by tests.
- rathole is a supervised child process: if it dies it restarts, instead of taking the
  container down with it.
- No behaviour change for the tunnel itself.

## 1.1.1

- Guard against an invalid connector assignment (missing `service`, `token`
  or `server`) coming from either a fresh registration or a saved state
  file. Instead of writing a broken `rathole-client.toml` and crashing on
  parse, the add-on now clears the saved state and fails with a clear error
  so the next restart registers from scratch.

## 1.1.0

- Persist the connector assignment (`service`, `token`, `server`) returned by
  the Gateway on first registration. On subsequent starts (add-on restart, HA
  reboot, etc.) the saved assignment is reused instead of registering again,
  which previously failed with a `409 Unit connector already active` and
  produced an invalid `rathole-client.toml`.

## 1.0.0

- Initial release.
