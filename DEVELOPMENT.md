# Local development with Docker

This fork uses TriliumNext's source editor, including its built-in multicolumn
layout. No additional editor plugin or storage format is needed.

## Start and stop

From the repository root, with Docker running and Compose 2.32 or newer:

```sh
docker compose -f compose.dev.yaml up --build --watch
```

Open <http://localhost:7772>. The first start installs the exact dependencies
from the lockfile and prepares the client assets; subsequent starts reuse the
Docker volumes. Complete Trilium's setup with a development password and choose
a fresh database. This instance uses its own `trilium-dev_data` volume. It never
mounts the database used by another Trilium installation.

Compose Watch syncs this checkout into the Linux container. This avoids macOS
bind-mount permission errors during Node asset copying. The server watches
TypeScript changes, and Vite serves the client in development mode. Keep the
watch command running while developing; package-manager changes rebuild the
container. Refresh the browser after editing editor plugins if hot reload cannot
apply the change. Node and pnpm are pinned to
`.nvmrc` and the root `packageManager` respectively; update the Dockerfile when
those pins change. Electron is not downloaded for this browser-based environment.

```sh
# Stop the service and retain all development notes.
docker compose -f compose.dev.yaml stop

# Restart it.
docker compose -f compose.dev.yaml up --watch

# Run commands inside the same Linux environment.
docker compose -f compose.dev.yaml exec dev sh
```

To use another host port, prefix the start command with `TRILIUM_DEV_PORT=7773`.
Vite hot reload also uses localhost:8090. Only the local machine can access
the published ports. Do not use `down --volumes` unless you intend to delete this environment's development notes and caches.
Linux dependencies live in Docker; do not reuse them for native macOS commands.

## Two columns inside a text note

1. Create or open a **text** note and place the cursor in its content.
2. In the editor's **Insert** menu, choose **Columns** (two equal columns),
   or type `/columns` and select **2 columns layout**.
3. Click inside either column to edit it. Both columns belong to the same note
   and save through the usual note autosave.

The contextual toolbar changes column widths or removes the layout while keeping
its content. Columns stack vertically when the available width is below 500 px.
The feature is already part of the fork's upstream base; its implementation is in
`packages/ckeditor5/src/plugins/multicolumn`, with styling in
`packages/ckeditor5/src/theme/multicolumn.css` and editor menu integration in
`apps/client/src/widgets/type_widgets/text/toolbar.ts`.

See the [Multicolumn layout guide](docs/User%20Guide/User%20Guide/Note%20Types/Text/Multicolumn%20layout.md)
for keyboard shortcuts, printing, and export behavior.

## Verify the editor

Install the test browser and its system libraries once per tooling container:

```sh
docker compose -f compose.dev.yaml exec dev pnpm exec playwright install --with-deps chromium
```

Run the existing browser-based multicolumn tests:

```sh
docker compose -f compose.dev.yaml exec dev pnpm --filter @triliumnext/ckeditor5 test src/plugins/multicolumn
```

The browser download is cached in a Docker volume. System libraries must be
installed again after recreating the tooling container. Test output remains
inside the container. Copy it out when needed:

```sh
docker compose -f compose.dev.yaml cp dev:/workspace/packages/ckeditor5/test-output ./test-output
```
