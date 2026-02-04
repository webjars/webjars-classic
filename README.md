WebJars Classic
---------------

This is the configuration repository for Classic WebJars. Classic WebJars are created from GitHub releases or NPM packages.

For more information about WebJars visit the website: http://www.webjars.org


## Format

Each WebJar is defined by a `.properties` file. There are two types of configurations:

### GitHub-based WebJars

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Display name of the WebJar |
| `repo` | Yes | GitHub repository in `owner/repo` format |
| `download` | No | Custom download URL (supports `${version}` placeholder). If not specified, uses GitHub release archives. |
| `base.dir` | No | Base directory within the archive to extract (e.g., `*/dist`, `package/build/`) |
| `requirejs.main` | No | Main file for RequireJS configuration |

Example:
```properties
name=Swagger UI
repo=swagger-api/swagger-ui
requirejs.main=swagger-ui
base.dir=*/dist
```

With custom download URL:
```properties
name=Vega-Embed
repo=vega/vega-embed
download=https://registry.npmjs.org/vega-embed/-/vega-embed-${version}.tgz
base.dir=package/build/
```

### NPM-based WebJars

| Field | Required | Description |
|-------|----------|-------------|
| `npm` | Yes | NPM package name |
| `license.name` | No | License name (if not auto-detected) |
| `license.url` | No | License URL |

Example:
```properties
npm=some-npm-package
license.name=MIT
license.url=https://opensource.org/licenses/MIT
```


## Testing

Test a WebJar configuration by POSTing the properties file content to the WebJars API:

```
POST https://www.webjars.org/create/classic?nameOrUrlish=<name>&version=<version>
Content-Type: text/plain

<properties file content>
```

### Parameters
- `<name>` - the properties file name without the `.properties` extension
- `<version>` - a valid version from the source (GitHub release tag or NPM version)

### Response
- **Success**: HTTP 200 with `Content-Type: application/java-archive` (binary JAR file)
- **Failure**: HTTP 400 with `Content-Type: text/plain` containing an error message

### curl Examples

Test Swagger UI (version must match GitHub release tag, e.g., `v5.31.0`):
```bash
curl -X POST "https://www.webjars.org/create/classic?nameOrUrlish=swagger-ui&version=v5.31.0" \
  -H "Content-Type: text/plain" \
  --data-binary @swagger-ui.properties \
  -o swagger-ui.jar
```

Test HAL Explorer:
```bash
curl -X POST "https://www.webjars.org/create/classic?nameOrUrlish=hal-explorer&version=2.2.1" \
  -H "Content-Type: text/plain" \
  --data-binary @hal-explorer.properties \
  -o hal-explorer.jar
```

### AI Agent Validation

To programmatically validate a new properties file:

```bash
# Test the configuration
response_code=$(curl -s -X POST \
  "https://www.webjars.org/create/classic?nameOrUrlish=<name>&version=<version>" \
  -H "Content-Type: text/plain" \
  --data-binary @<name>.properties \
  -o /tmp/test.jar \
  -w "%{http_code}")

if [ "$response_code" = "200" ]; then
  echo "Success: WebJar built successfully"
  unzip -l /tmp/test.jar | head -20  # Verify JAR contents
else
  echo "Error: $(cat /tmp/test.jar)"  # Show error message
fi
```
