# PC

Client config endpoints:
- `https://clientconfig.rpg.riotgames.com/api/v1/config/{config_type}?namespace=keystone.products.valorant.patchlines`
- `https://clientconfig.esports.rpg.riotgames.com/api/v1/config/{config_type}?namespace=keystone.products.valorant.patchlines`

Valid `config_type` values:
- `public`
- `player` (requires auth: Bearer auth access token + `X-Riot-Entitlements-JWT`)

URL for manifests is under `{patchline-key}.platforms.win[].patch_url`. I now noticed that there is a new `patch_artifacts` property as well, unsure what it is, might worth keeping an eye on this too.

The `upload_timestamp` in `manifest.json` in this repo comes from the `Last-Modified` response header.

For PC builds the build info is extracted from the game executable (`VALORANT-Win64-Shipping.exe`); \
![image](https://github.com/user-attachments/assets/7c021559-3fe0-40f1-a508-30e09372d51a) \
Python script to achieve this: https://gist.github.com/floxay/a6bdacbd8db2298be602d330a43976da \
The `pefile` module is used as the build version is not always present, in that case it gets extracted from the VERSIONINFO resource (ProductVersion).

# Console
To my knowledge for console complete builds are not available at the moment so those cannot be tracked. \
For hotfixes console uses the Sieve API; \
URL: `https://sieve.services.riotcdn.net/api/v1/products/valorant/version-sets/{version_set}`

All the version sets I'm aware of: \
```py
class ValorantVersionSet(VersionSet):
    NA = auto()
    BR = auto()
    LATAM = auto()
    AP = auto()
    KR = auto()
    EU = auto()
    EXT = auto()
    EXT1 = auto()
    EXT2 = auto()
    SANDBOX = auto()
    SANDBOX1 = auto()
    STAGE2 = auto()
    CERT = auto()
    EXT1_NA1 = "ext1-na1"
```
Most of these are empty and out of the 6 live PC regions only 4 here are active currently.

Many values in here do not make sense to me, thankfully these console releases contain a file called `build.version` which holds information of the build which I prefer over the values in the Sieve API response: \
![image](https://github.com/user-attachments/assets/f5c00efd-0e79-4fcd-a7f7-a631ba29348c)

PS.: The entries with `scd_required` set to `true` means they require valid access for the Secure Content Distribution system. \
A signed CloudFront cookie can be acquired after `GET https://scd.riotcdn.net/api/v1/cookies` with `Authorization` (Bearer) and `X-Riot-Entitlements-JWT` present in the request. \
The response body contains the cookie string(s) with additional information on where they can be used.

# Manifest files
https://github.com/moonshadow565/rman is probably the most comprehensive project to work with Riot's release manifest files. \
`rman-dl` can be used to download files listed in the manifest, the bundle CDN URL should be changed, for public stuff it is currently `http://valorant.dyn.riotcdn.net/channels/public/bundles`.
