# knip-serverless-issue [resolved in knip@6.12.2]

Minimal reproduction for a bug in knip's Serverless Framework plugin: when a Lambda function uses a **container image** (no `handler` field), knip crashes with an uncaught error.

## The bug

Running `knip` against a `serverless.yml` that has a function **without a `handler`** produces:

```
ERROR: Error loading serverless.yml (Cannot read properties of undefined (reading 'lastIndexOf'))
ERROR: Please fix or visit https://knip.dev/reference/known-issues
```

`handler` is optional — the Serverless Framework supports deploying Lambda functions via an ECR container image, in which case `handler` is omitted and an `image` block is used instead.

## Reproduce

```bash
npm install
npm run lint   # → crashes with the error above
```

## What works vs. what doesn't

### Does NOT work — image-based function (no `handler`)

`serverless.yml` as-is in this repo:

```yaml
functions:
  my-function:
    timeout: 900
    memorySize: 256
    image:
      name: sls-service
      command: dist/functions/my-function.handler
```

knip output:

```
ERROR: Error loading serverless.yml (Cannot read properties of undefined (reading 'lastIndexOf'))
ERROR: Please fix or visit https://knip.dev/reference/known-issues
```

### Works fine — handler-based function

Uncomment the `handler` line in `serverless.yml`:

```yaml
functions:
  my-function:
    timeout: 900
    memorySize: 256
    handler: users.create # ← add this
    image:
      name: sls-service
      command: dist/functions/my-function.handler
```

knip runs without the crash (though `handler` is semantically incorrect for image-based functions).

## Root cause (hypothesis)

Knip's Serverless plugin reads `function.handler` and calls `.lastIndexOf('.')` on it to split the module path from the export name. When `handler` is `undefined` (image-based function), this throws a `TypeError` instead of being skipped gracefully.

## Environment

- knip: `^6.12.1`
- serverless: `^4.35.1`
