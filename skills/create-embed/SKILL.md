---
name: create-embed
description: "Create or update Hubble iframe HTML embeds: workspace-local mini
  apps authored as .html files and embedded from Markdown with iframe src."
---
# Create Embed

Build a workspace-local `.html` mini app, then embed it from Markdown:

```
<iframe src="./file-index.html"></iframe>
```

The `src` must be a relative workspace-local `.html` path from the Markdown file.

## Runtime

Hubble provides the embed runtime. Write plain HTML that assumes these globals are already available:

- `hubble`
- Alpine
- Tailwind browser v4
- Hubble theme tokens

Use Alpine and Tailwind by default. Do not add dependency `<script>` tags or set up package files, lockfiles, or `node_modules` for an embed.

Available runtime APIs:

```
await hubble.files.list("**/*.md");
await hubble.files.read("path/inside/workspace.md");
```

## Template

```
<!doctype html>
<html lang="en">
	<head>
		<meta charset="utf-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1" />
		<title>Workspace files</title>
	</head>
	<body class="m-0 bg-background p-3 font-sans text-foreground">
		<main
			class="grid gap-3 rounded-md border border-border bg-card p-3 text-card-foreground"
			x-data="{
				files: [],
				error: '',
				async init() {
					try {
						this.files = await hubble.files.list('**/*.md')
					} catch (error) {
						this.error = error.message || 'Could not load files'
					}
				},
			}"
		>
			<header class="flex items-center justify-between gap-3">
				<h1 class="m-0 text-base font-semibold">Workspace files</h1>
				<span class="text-sm text-muted-foreground" x-text="`${files.length} files`"></span>
			</header>

			<p class="m-0 text-sm text-muted-foreground" x-show="!error && files.length === 0">
				Loading files...
			</p>
			<p class="m-0 text-sm text-destructive" x-show="error" x-text="error"></p>

			<ul class="m-0 grid list-none gap-2 p-0" x-show="!error && files.length > 0">
				<template x-for="file in files" :key="file.path">
					<li class="rounded-sm border border-border bg-background px-3 py-2">
						<span class="text-sm font-medium" x-text="file.path"></span>
					</li>
				</template>
			</ul>
		</main>
	</body>
</html>
```

## Native Feel

Use Hubble tokens through Tailwind classes so the embed feels native:

- Surfaces: `bg-background`, `bg-card`, `bg-popover`
- Text: `text-foreground`, `text-muted-foreground`, `text-card-foreground`
- Borders/rings: `border-border`, `border-input`, `ring-ring`, `focus-visible:ring-ring`
- Actions: `bg-primary text-primary-foreground`, `bg-secondary text-secondary-foreground`
- States: `hover:bg-accent`, `bg-selected text-selected-foreground`

Keep spacing compact: `gap-2`, `gap-3`, `p-2`, `p-3`, `px-3`, `py-2`. Use `rounded-sm` or `rounded-md` for controls.

Avoid one-off palettes, oversized cards, hero layouts, and decorative gradients unless the user asks for a themed visual.

## References

Read only the patterns you need:

- [Buttons](references/buttons.md)
- [Radio selects](references/radio-selects.md)
- [Tabs](references/tabs.md)
- [Forms](references/forms.md)
- [Lists and empty states](references/lists-and-empty-states.md)
