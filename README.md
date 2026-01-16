# z-base

z-base is a zero-knowledge state relay and persistence layer designed to make privacy-preserving web applications the default, not an afterthought.

It enables rich, real-time, offline-first applications without requiring service providers to process, store, or understand users’ personal data.

---

## Core Value Propositions

### For service providers

z-base drastically reduces liability by removing the need to process, inspect, or manage users’ personal or organizational data.  
You operate infrastructure, not people’s private lives.

### For end users

Users get powerful information and communication features without surveillance, profiling, or hidden data extraction.  
Their data remains private, local-first, and under their control.

### For platform builders

z-base removes the ethically compromised role of becoming a custodian of millions of users’ daily activities.  
You provide services, not dossiers.

### For developers

The model maps cleanly to familiar real-time object storage and synchronization patterns.  
The API surface is intentionally small, semantically consistent, and well documented, making implementation straightforward.

---

## Where to Use z-base

Use z-base as the **user or organization data layer** of your application.

It is well suited for:
- user-generated or organization-generated content,
- personal or collaborative data,
- state that should morally and practically be owned by its creator,
- systems where privacy, offline access, and peer verification matter.

In short: use z-base wherever user data should remain under user control.

---

## Where *Not* to Use z-base

Do **not** use z-base when:
- scheduled or unattended automation must modify data without user presence,
- third-party systems need unrestricted access to plaintext data,
- public discovery or indexing of content is a core requirement.

In these cases, use **separate, purpose-built databases** for automation, analytics, or public content.

If automation is required, design it ethically:
- request **explicit, verifiable permission** from the user,
- limit access to a **narrow data scope**,
- enforce a **bounded time window**,
- state a **specific purpose** aligned with your service terms,
- make all data exposure visible and measurable in the UI.

---

## Philosophy

z-base exists to make it easy to build ethical services.

If your primary goal is maximizing profit through surveillance, data brokerage, or opaque advertising ecosystems, z-base is probably not a fit.

If you still want to monetize data, consider doing so transparently and consensually—by offering to **purchase clearly defined data sets directly from users**, under terms they understand and control.

This is infrastructure for a web that serves people first.
