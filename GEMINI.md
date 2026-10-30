# Amplify

The `amplify` MCP server works in the user's own Amplify account (amplify.is): it builds and edits browser games and web apps.

- Building and editing spend the user's credits. `create_project`, `create_build` and `edit_project` first return `needs_confirmation` with a price and a `confirmToken`. Show the price to the user and call `confirm_action` only after they say yes.
- `publish_project` only queues an approval. Nothing goes live until the owner approves it in the Amplify app (Approvals tab).
- Use `get_build_status` to follow a build and `get_result` for the preview and claim link.
