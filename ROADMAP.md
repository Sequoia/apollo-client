# Apollo Client Roadmap

## Product vision

Apollo Client is a framework for consuming a data graph and binding it to a user interface. It is written in TypeScript, designed to work with React, and integrates with the Apollo GraphQL Platform. Apollo Client is easy to use, incrementally adoptable into a React/Redux application, fully documented from day one to advanced abilities, and scales to application needs at enterprise scale. Our goal is to not only make Apollo Client the *best way to use GraphQL, but the best way to manage data for React apps*. Apollo Client also ships with a first class set of developer tools including a CLI, VS Code extension, and in browser debugging tools.

## Apollo Client 3.0

**Completion date:** Full RC available end of October 2019

The next version(s) of Apollo Client (roughly referred to as Apollo Client 3) is a pruning of outdated and overlapping API surface areas, a redesign of the core caching layer, and a simplification of the packaging of the project.

### Goals

* Only one package is required to install and use all core Apollo Client features (including React integration and networking), for teams of all sizes
  * This helps make documentation easier to understand, and helps get developers to their first “feature shipped” without having to learn multiple packages
  * This also helps teams customize their build of Apollo Client for advanced usages

* The default `@apollo/client` bundle is 60kb minified (but not gzipped) if using no build tools (less if using build tools)
  * This is roughly half the size of the current libraries and half the size of `react` + `react-dom`
  * This includes all packages needed for an out of the box full Apollo Client experience (Apollo Client core with cache and utilities, React hooks integration, and networking). It will be possible to enable/disable features to reduce the overall bundle size

* Performance: using a 4x CPU slowdown in Chrome, Apollo Client’s fetch and cache is at or less than 16ms to account for repaint frame rates
  * This is a rough approximation of mid range mobile devices

* The new `EntityCache` (name TBD) furthers the ideas started with federation to take advantage of a principled datagraph
  * Uses entities and keys to know object identity for selective normalization and better performance
  * Works well for SSR and offline apps including storing of errors and priority rendering given to prefetched work
  * Has an easy to understand config that can be generated by the CLI but is clear and understood by teams writing the config manually

* Fully redesigned documentation that helps teams be productive throughout their Apollo Client adoption
  * Features a new getting started experience targeted for React engineers to be productive from day one
  * Explains the architecture and cache design more clearly as teams move onto more advanced features

### How we get there

Apollo Client 3 will be a rewrite in place of the Apollo Client project, while we support and maintain Apollo Client 2.x in a reduced manner. Since we are moving to a new package name and structure, as soon as the new core is usable by production teams, we will begin an alpha cycle to get concrete performance numbers from real world usage.

Apollo Client 3 will feature simplified packaging for easier use and smaller bundles, and a new cache implementation with more control and better performance. The work to accomplish these goals is being grouped as follows:

* **Simplified packaging for easier use and smaller bundles:**

  * Two packages (down from the current ~15 packages): `@apollo/client` and `@apollo/graphql`
  * Separate reduced and focused bundles will be available to allow applications to use only the parts of Apollo Client they want to (e.g. `@apollo/client/core` for just the AC core + cache)
  * Further bundle size reductions by pruning `apollo-utilities` and other API cruft needed to support the generic APIs currently supported
  * Reduce mental overhead for teams by giving them a “boost” like experience built into the single package that also works with React out of the box. More advanced teams use the same package by building their own version if needed using file imports
    * This follows the same pattern that `@apollo/graphql` and `@apollo/server` will use
  * Prune core client API to reduce confusion, increase performance, and thin bundle size
    * Remove `watchQuery` in favor of a single query method that returns a watchable result stream
    * Remove custom cache implementation entry point in favor of a simplified config API that can be generated by build tools
    * Remove multiple “store reset” APIs that are confusing and overlap
    * Remove costly deep equality checks in favor of immutable structures
  * Avoid making subscriptions part of the default build of Apollo Client, but instead an additional part that can be imported and used
  * Instead of GraphQL ASTs, the cache will be powered by an efficient “IR” (intermediate representation) that can be understood both at runtime or at build time for teams that want to save extra performance
    * This allows teams to completely remove `graphql-js` and startup parsing work from the bundle and ship less bytes over the wire with a predefined set of cache actions per operation/fragment

* **New cache implementation with more control and better performance:**

  * Improve performance of pre-rendered datasets done from SSR or offline storage so that the UI is painted immediately before the data is linked in the cache. This removes blocking thread work for teams that have already fetched the data leading to faster paints.
  * Support custom normalization through a new config API. Custom normalization allows for parts of a response to skip costly normalization when it is not needed by the UI. This will use the concept of `Entities` from Apollo Server and allow for teams to tailor performance of the cache to their specific UI needs. Put another way, the new cache will support both the normalization and de-normalization of stored data.
    * This allows teams with very large datasets to opt out of more expensive work resulting in much faster reads from the cache
    * It also makes for a more predictable set of actions for working with the cache and enables more possibilities around using fragments directly
  * Automatically garbage collect orphaned entities in the cache to reduce cache memory footprint and “unbound growth” problems for long running applications
  * Properly support cache eviction so that mutations can clean up entities in the cache and data that is deemed “too old” can be pruned from the cache. This allows long running applications to keep the cache small and snappy
  * Add support for storing errors in the cache. This is critical for offline and SSR apps that may encounter errors which are rendered into the UI.

* **Completely re-architected documentation site explaining the value of Apollo Client:**

  * Features a new how to get started section targeted for professional React developers to be productive from day one
  * Better explains the architecture of the client and its core features
  * Includes in depth guidelines for running in production, with build tooling (CLI/TypeScript) and better recipes around common actions like auth

## After Apollo Client 3.0

While we don’t have concrete timelines for the next set of work, we think these are the next challenges to tackle ordered by priority:

* Redesign Apollo Client's network layer
  * We're planning out a simplified but flexible network layer that removes `Observable`'s from the `ApolloLink` API in favor of language primitives. This will simplify custom building, trim bundles, and better support multiple responses.
* Supporting `@defer` and `@stream`
* Fragment based API for component isolation
* Improved local state API
* Cache segmenting
