# cicd-demo
My first CI/CD Pipeline

A minimal Python (Flask) web service that demonstrates a complete CI/CD
pipeline with GitHub Actions, container publishing to GitHub Container
Registry (GHCR), and a security gate.

## The application
A tiny Flask app with two endpoints:
- `GET /`        a JSON greeting
- `GET /health`  a JSON health check

It runs under gunicorn inside a container that runs as a non-root user.

## How the pipeline works
On every push or pull request to `main`:
1. **test** installs dependencies, lints with ruff, and runs pytest.
2. **build-scan-push** (only if tests pass): builds the image, scans it with
   Trivy and fails on HIGH/CRITICAL fixable vulnerabilities, then pushes to
   GHCR on pushes to `main`. The scan runs before the push, so a vulnerable
   image is never published.

## Reproduce it
The whole pipeline runs automatically in GitHub Actions when you push.
To run the published container yourself (requires Docker):
    docker run -p 8000:8000 ghcr.io/<your-username>/cicd-demo:latest

## Security choices and reasoning
- Container image scanning (Trivy) is the main gate, because the deployed
  artifact is the image: it bundles the OS, runtime, and dependencies, which
  is where most vulnerabilities live. We scan the exact thing we ship.
- Least-privilege token: read-only by default, packages:write only where needed.
- No long-lived secrets: publishing uses the built-in GITHUB_TOKEN.
- Non-root container user.
- Pinned dependencies for reproducible builds.
- GitHub secret scanning + Dependabot enabled to cover committed secrets and
  dependency drift over time.
