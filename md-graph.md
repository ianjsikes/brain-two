# md-graph

[[computers]]

## Notes

- Wiki Links do not resolve with subfolders. Only foam generated linkrefdefs work.
  The wiki links will work fine if both files are in the same folder.

  **Solution:** Detect the presence of the foam extension. If not installed,
  use this default (broken) behavior. If Foam _is_ installed, skip wiki link parsing
  and just parse Foam's linkrefdefs instead.

  **Better Solution:** Use Foam's note graph directly to avoid double parsing the
  workspace. This could be done in two ways:

  - Add a dependency for foam-core. This would still be redundant parsing, but it would
    have the same logic as Foam for consistency. This could be done today.
    Relevant: [File parsing code](https://github.com/foambubble/foam/blob/3e084fd944bb07027281a4e2a22468381474c467/packages/foam-core/src/note-graph.ts#L66)
  - Use some sort of inter-extension communication to share a single graph. I think
    the implementation of this is planned for Foam but still WIP, so its not ready yet.

- Related to the point above, explore using [graphlib](https://github.com/dagrejs/graphlib)
  instead of custom graph implementation. Less code to write/maintain, plus it supports some
  nice features. It is also the same lib used by Foam, so it could make interop easier.

[//begin]: # "Autogenerated link references for markdown compatibility"
[computers]: computers "Computers"
[//end]: # "Autogenerated link references"
