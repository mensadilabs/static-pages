---
authors: [Immich Team]
description: A review of the changes in Immich v3, including a list of breaking changes.
id: 85bf6ab2-95b2-4c67-96b2-5a391e4a5af9
publishedAt: 2026-06-01
slug: v3-release
title: Immich v3
---

Welcome to Immich `v3`!


## Breaking changes


:::tip
Found something missing? Let us know and we'll get it fixed.

:::

This release includes a hefty list of breaking changes, most of which were previously deprecated. This post aims to document, explain, and guide users as they upgrade to `v3`.

### Mobile timeline

The legacy timeline in the mobile app has been removed.


:::info
If you still experience issues with the new timeline, please open an issue on [GitHub](https://github.com/immich-app/immich/issues/new/choose) so we can take a look.

:::

### Environment variables

#### `IMMICH_MACHINE_LEARNING_PING_TIMEOUT`

The environment variable `IMMICH_MACHINE_LEARNING_PING_TIMEOUT` has been removed. Similar functionality can be achieved through the `machineLearning.availabilityChecks.timeout` admin settings.

### Server jobs

#### Removed `AuditLogCleanup` job

The `AuditLogCleanup` job has been removed.



:::info
This release contains a lot of changes to the Immich API. Many deprecated endpoints have been removed. Also, some request/response schemas have been updated. See the full details below.

:::

### Response DTOs

#### `AssetMediaCreateDto`

* The `deviceId` and `deviceAssetId` properties of `AssetMediaCreateDto` have been removed.

#### `AssetResponseDto`

* The `deviceId` and `deviceAssetId` properties of `AssetResponseDto` have been removed.

#### `SharedLinkResponse`

The `token` response property of `SharedLinkResponse` has been removed.

### Error responses

Previous to `v3`, the server used `class-validator` for request validation, but starting with `v3`, the server has migrated to [Zod](https://zod.dev/) and now sends back a different error response object. Additionally, the `correlationId` response property has been migrated to the `X-Correlation-ID` response header.

#### Old structure

```typescript
{
  "message": [
    "[comment] Comment must not be provided when type is not COMMENT"
  ],
  "error": "Bad Request",
  "statusCode": 400,
  "correlationId": "dtw6imvq"
}
```

#### New structure

```typescript
{
  "message": "Validation failed",
  "errors": [
     {
        "path": ["comment"],
        "message": "Comment is required when type is COMMENT",
    },
  ]
}
```

#### Error messages

Some error messages have been updated to avoid leaking resource existence or permission details. For more details see [#28154](https://github.com/immich-app/immich/pull/28154).

### Endpoints (removed)

#### `replaceAssest`

The `replaceAsset` endpoint (`PUT /assets/:id/original`) has been removed. The API Key permission for this endpoint, `asset.replace`, has also been removed.


:::info
Use `copyAsset` (`PUT /assets/:id/clone`) instead for similar functionality.

:::

#### `getRandom`

The `getRandom` endpoint (`GET /assets/random`) has been removed.


:::info
Use `searchRandom` (`POST /search/random`) instead for similar functionality.

:::

#### `getDeltaSync`

The `getDeltaSync` endpoint (`POST /sync/delta-sync`) has been removed.

#### `getFullSyncForUser`

The `getFullSyncForUser` endpoint `POST /sync/full-sync` has been removed.

#### `checkExistingAssets`

The `checkExistingAssets` endpoint (`POST /assets/exists`) has been removed.

#### `getAllUserAssetsByDeviceId`

The `getAllUserAssetsByDeviceId` endpoint (`GET /assets/device/:deviceId`) has been removed.

### Shared links access

#### Add to shared link

Previous to `v3` adding an asset to a shared link (without being logged in) required two API requests: one to upload the asset and another one to add it to the shared link. Now, assets are automatically added to the associated shared link when they are uploaded, removing the need to shared link access to the following APIs, which have been removed:

* `addAssetsToAlbum` (`PUT /albums/:id/assets`)
* `addAssetsToAlbums` (`PUT /albums/assets`)
* `addSharedLinkAssets` (`PUT /shared-links/:id/assets`)

#### Shared link login

Endpoints that require shared link authentication now no longer accept `query.password`. Instead, it should be send as `body.password` in the `sharedLinkLogin` (`POST /shared-links`) endpoint to login, receive a cookie, and use that with subsequent requests.