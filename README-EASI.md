
# EASI notes for TerriaMap fork

## What is EASI-specific in this fork (keep these on upstream merges)

Keep this repo as a small overlay on upstream TerriaMap. The key EASI-specific differences are:

| Change | File | Description |
|--|--|--|
| Add trusted server for EASI/ADIAS integration | `index.js` | `TrustedServers.add("insitu.adias.aquawatchaus.space", "443")` |
| Dynamic disclaimer support | `lib/Views/showGlobalDisclaimer.ts` | Use `globalDisclaimer.dynamic` when configured, otherwise falls back to default text |
| EASI deployment path/documentation | `README-EASI.md` | Docker/ECR workflow for EASI image build and publish |
| EASI runtime/dependencies | - `serverconfig.json` | - simpler proxy list |

Everything else should stay as close to upstream as possible (or be documented here as an exception).

## Upstream merge recipe

Use this whenever syncing from TerriaJS/TerriaMap into this fork.

```bash
git clone -b easi-develop https://github.com/csiro-easi/TerriaMap.git
cd TerriaMap

git checkout -b chore/sync-upstream-vX.Y.Z
git remote add upstream git@github.com:TerriaJS/TerriaMap.git
git fetch upstream --tags
git merge upstream/main
```

Conflict resolution rule:

- Prefer upstream by default.
- Re-apply/retain only the EASI-specific differences listed above.
- For `yarn.lock`: usually take upstream lockfile as the base, then verify.

`yarn.lock` resolution and validation:

```bash
# while resolving merge conflicts
git checkout --theirs yarn.lock
git add yarn.lock

# verify lockfile matches manifests without rewriting it
yarn install --frozen-lockfile
```

If frozen install fails due to retained local manifest deltas:

```bash
yarn install
git add yarn.lock
```

Final validation after merge:

```bash
yarn build
yarn audit --groups dependencies
# optional alternative in some environments:
npm audit --omit=dev
```

Optional clean-environment check:

```bash
docker build -t easi-terriamap .
```

## Build Docker image

```bash
git clone -b easi-develop --depth 1 https://github.com/csiro-easi/TerriaMap.git
cd TerriaMap
docker build -t easi-terriamap .
```

## Push image to ECR Global (EASI-CSIRO)

ECR repository must exist before push.

```bash
export AWS_ACCESS_KEY_ID=$(AwsAccessKeyID)
export AWS_SECRET_ACCESS_KEY=$(AWSSecretAccessKey)
export AWS_SESSION_TOKEN=$(AWSSessionToken)

aws ecr get-login-password --region=ap-southeast-2 | docker login --username AWS --password-stdin 444488357543.dkr.ecr.ap-southeast-2.amazonaws.com
docker tag easi-terriamap:latest 444488357543.dkr.ecr.ap-southeast-2.amazonaws.com/easi-terriamap:$(TAG)
docker push 444488357543.dkr.ecr.ap-southeast-2.amazonaws.com/easi-terriamap:$(TAG)
```
