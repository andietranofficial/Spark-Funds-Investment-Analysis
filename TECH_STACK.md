# Tech Stack

This is a **Microsoft Fabric App**: a React and TypeScript frontend built on Rayfin, Microsoft's app framework for Fabric. It's set up as an npm monorepo.

## Frontend

- **Core:** React 19 and TypeScript 5.7
- **Build:** Vite 8 with the SWC React plugin
- **Styling:** Tailwind CSS v4, with `clsx` and `tailwind-merge` for class handling
- **UI:** Radix UI (dialogs), Lucide icons and Framer Motion for animation
- **Charts and grids:** `@microsoft/fabric-visuals` and `@microsoft/fabric-datagrid`, plus Vega-Lite (dev dependency)

## Platform (Rayfin, `1.36.0-alpha`)

- **Client and data:** `rayfin-client`, `rayfin-data`, `rayfin-core` and `rayfin-lib`
- **Tooling:** `rayfin-cli` and `rayfin-local-dev`
- **Sign-in:** Microsoft Fabric/Entra authentication, with password login turned off

## Hosting and services

From [rayfin.yml](rayfin/rayfin.yml):

| Service | Status |
| --- | --- |
| Authentication | On |
| Static hosting | On, with protected assets |
| Data | Off |
| Storage | Off |
| Functions | Off |

The config lists a Fabric web app URL in Switzerland North as an allowed sign-in redirect.

## Testing and code quality

- **Tests:** Vitest, Testing Library and jsdom
- **Linting:** ESLint with `typescript-eslint` and the React hooks plugins

## Current scope

So far, only the `visuals` capability pack has been added. The app has no database or backend functions turned on, and no packs for semantic models or other data connectors. Those can be added later as packs.
