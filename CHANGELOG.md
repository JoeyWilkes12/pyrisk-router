# Changelog

All notable changes to this router are documented here. The format follows
[Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/) and releases follow
[Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html).

Patch releases fix copy, metadata, or documentation. Minor releases introduce a
new destination or added checks. Major releases change the public address or
QR code, which should never happen: both are intended to be permanent.

## [1.0.0] - 2026-10-28

### Added

- Permanent PyRisk router address and PNG/SVG QR assets for printed and shared links.
- One Liquid destination setting for JavaScript and HTML redirects, the fallback
  link, canonical URL, and educational WebApplication metadata.
- Accessible fallback page, crawler files, local preview and regression scripts.
- Maintenance instructions, agent guidance, and router versioning.

The initial destination is the Vercel Production address. It is not live yet;
external destination validation is expected to fail until deployment.
