# GitHub Action &ndash; Tag and Release (New)

Automatically create tags and corresponding releases.

Based on [avakar/tag-and-release](https://github.com/avakar/tag-and-release).

## Usage

This action is meant to be invoked in response to a branch push to create
a tag and a corresponding release, under the assumption that you can derive
the tag name automatically.
In contrast,
[`actions/create-release`](https://github.com/actions/create-release)
is generally run on a tag push, expects the tag to already exist
and only creates the release.

The only mandatory input parameter is the tag name.
```yaml
name: Create Release
on:
  workflow_dispatch:
    inputs:
      tag_name:
        description: 'The name of the tag to create'
        required: true
        default: 'v1.0.0'
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: BrandonStudio/tag-and-release@v1
        with:
          tag_name: ${{ inputs.tag_name }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

- `github_token`: the GitHub token to use for authentication
- `tag_name`: the name of the tag to be created
- `commit`: the commit to which the new tag should point, defaults to `${{ GITHUB_SHA }}`
- `release_name`: the name of the new release; if omitted, defaults to `tag_name`
- `body`: optional text of the release body
- `draft`: set to `true` to create an unpublished (draft) release; defaults to `false`
- `prerelease`: whether the release should be marked as a prerelease.
- `discussion_category_name`: If specified, a discussion of the specified category is created and linked to the release. The value must be a category that already exists in the repository. For more information, see [Managing categories for discussions in your repository](https://docs.github.com/discussions/managing-discussions-for-your-community/managing-categories-for-discussions-in-your-repository).
- `generate_release_notes`: Whether to automatically generate the name and body for this release. If `name` is specified, the specified name will be used; otherwise, a name will be automatically generated. If `body` is specified, the body will be pre-pended to the automatically generated notes.
- `make_latest`: Specifies whether this release should be set as the latest release for the repository. Drafts and prereleases cannot be set as latest. Defaults to `true` for newly published releases. `legacy` specifies that the latest release should be determined based on the release creation date and higher semantic version.


### Outputs

- `id`: the ID of the release
- `html_url`: the human-readable web-page of the release, e.g. `https://github.com/BrandonStudio/tag-and-release/releases/v1.0.0`
- `upload_url`: the upload URL.
- `discussion_url`: The URL of the release discussion.

### Tips
The `upload_url` may be like 
```
https://uploads.github.com/repos/BrandonStudio/tag-and-release/releases/v1/assets{?name,label}
```
or
```
https://uploads.github.com/repos/BrandonStudio/tag-and-release/releases/v1/assets?name
```
It is easy to unify to `assets?name` to upload assets.

**Ubuntu & macOS**
```bash
transformerd_url=$(echo $upload_url | sed 's/assets\(?!.*assets\).*/assets?name/')

```

**Windows**
```powershell
$transformed_url = [regex]::Replace($env:upload_url, "assets(?!.*assets).*", "assets?name")
```
Where `$upload_url` / `$env:upload_url` may be replaced by `${{ steps.<>.outputs.upload_url }}`.


Then you can use the `$transformed_url` to upload assets.

**Ubuntu & macOS**
```bash
curl -X POST -H "Authorization: token $GITHUB_TOKEN" \
-H "Content-Type: application/octet-stream" \
--data-binary @"$file" \
"$transformed_url=$(basename $file)"
```

**Windows**
```powershell
Invoke-RestMethod -Method Post `
-Uri "$transformed_url=$(Split-Path $file -Leaf)" `
-Headers @{"Authorization"="token $env:GITHUB_TOKEN"; "Content-Type"="application/octet-stream"} `
-InFile $file
```
Where `$file` is the path to the asset file you want to upload, may be within a for loop.
