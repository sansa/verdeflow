# VerdeFlow website

The research website for VerdeFlow, built with HTML, CSS, and JavaScript. No package installation or frontend framework is required.

## Source layout

```text
src/
  index.html               Page content, links, and slideshow panels
  styles.css               Responsive styles and motion
  app.js                   Slideshow, workflow, chart, and citation interactions
  assets/verdeflow-mark.png Existing VerdeFlow logo
  verdeflow.bib             Downloadable publication citation
scripts/build.mjs          Generates the deployment directory
.openai/hosting.json       Existing Sites identity and hosting configuration
```

Edit files in `src/`. The generated `dist/` directory is ignored by Git and recreated by the build command.

## Run locally with Docker Compose

Requires Docker with the Compose plugin. From `verde-flow-website/`, run:

```sh
docker compose up -d
```

Open http://localhost:4173. Nginx serves `src/` through a read-only bind mount, so source edits are visible after refreshing. No Node.js installation or build is needed for this mode. The port is bound to localhost.

If port 4173 is occupied:

```sh
WEBSITE_PORT=4174 docker compose up -d
```

Then open http://localhost:4174.

```sh
docker compose logs -f website  # View logs
docker compose down            # Stop and remove the container
```

## Develop

Requires Node.js 18+ and Python 3 for the local HTTP server.

```sh
npm run dev
```

Open http://127.0.0.1:4173. This serves `src/` directly; refresh after edits.

## Check, build, and preview

```sh
npm run check
npm run build
npm run preview
```

The build copies the complete source into `dist/`, including assets and the BibTeX download. The preview serves that generated directory at the same local address. Stop the development server before starting the preview server.

## Production Docker image

From `verde-flow-website/`, build a self-contained image:

```sh
docker build -t verdeflow-website:latest .
```

The build stage checks the JavaScript and generates `dist/` from `src/`. The runtime stage contains Nginx and the generated site, so it needs no source mount, Node.js, Python, or separate build step at deployment time.

Test the production image locally:

```sh
docker run --rm --name verdeflow-website-prod -p 127.0.0.1:8080:80 verdeflow-website:latest
```

Open http://localhost:8080. The image serves HTTP on container port 80 and includes a health check. Configure your deployment platform to route traffic to that port; use the platform’s ingress or reverse proxy for HTTPS.

To publish the image to your registry, replace `YOUR_REGISTRY` with your registry namespace:

```sh
docker tag verdeflow-website:latest YOUR_REGISTRY/verdeflow-website:latest
docker push YOUR_REGISTRY/verdeflow-website:latest
```

Rebuild the image after changing source files. Local `compose.yaml` remains the development setup that serves `src/` directly. `.dockerignore` excludes generated output, local environment files, and Sites metadata from the image build context.

## Hosting

Deploy the contents of `dist/`. The existing Sites manifest already points to this directory. Preserve its project ID for future updates. The Site is registered as private; a successful hosted deployment has not yet been completed.

## Content and interactions

The hero slideshow covers commit trends, connected devices, measurement methods, and the profiling workflow. It includes manual controls, pause/resume, and reduced-motion support. All displayed measurements and commit hashes are illustrative.

Device-agnostic measurement content reflects the project owner's description. Measurement interfaces and instruments can cover different scopes; the site retains that distinction.

The publication links use the GREENS 2026 record and DOI:
https://doi.org/10.1145/3786148.3788637

Repository and documentation links point to https://github.com/datasling/verdeflow. Public access was not confirmed, so the site labels the open-source release as in preparation. Update that copy once the repository is public.

Keep the citation in `src/index.html` synchronized with `src/verdeflow.bib` when editing publication metadata.

## Search discoverability

The production canonical domain is `https://verdeflow.org/`. HTML includes a descriptive title and description, social sharing metadata, and JSON-LD describing the website, software source, and related publication. Visible copy describes green coding and software energy measurement.

`src/robots.txt` permits crawling and references `src/sitemap.xml`. Both are copied into the production image by the existing build. The sitemap lists the single homepage; section fragments are not separate pages.

After deploying the updated image, verify HTTPS and public access to the homepage, `/robots.txt`, and `/sitemap.xml`. Add the domain to Google Search Console, complete its ownership verification, submit `https://verdeflow.org/sitemap.xml`, and request indexing of the homepage. Configure your ingress to redirect HTTP and the www alias to `https://verdeflow.org/`. Update canonical, social, JSON-LD, robots and sitemap URLs together if the primary domain changes.

Metadata helps search engines interpret content but does not guarantee indexing or ranking. No meta-keywords tag, invented reviews, or ratings are used.

## Kubernetes deployment

`depl.yaml` follows the existing Fantanka website deployment pattern: one Deployment, a Service, and an Nginx Ingress with a cert-manager certificate for `verdeflow.org` and `www.verdeflow.org`. The static website container listens on port 80. Readiness and liveness probes check the homepage.

Prerequisites: an Nginx Ingress controller, cert-manager with the existing `letsencrypt-prod` ClusterIssuer, DNS pointing at the ingress public IP, and access to the image in Docker Hub. If the registry repository is private, configure an image pull secret for your namespace and reference it through the workload's ServiceAccount or `imagePullSecrets`.

Publish the built image, then apply the manifest in your intended Kubernetes context and namespace:

```sh
docker push datasling/verdeflow:website-latest
kubectl apply -f depl.yaml
kubectl rollout status deployment/verdeflow-website-depl
```

The manifest uses the current kubectl namespace. Add `-n YOUR_NAMESPACE` to the kubectl commands if deploying into a specific existing namespace.

After pushing a new image under the same tag, restart the deployment to pull the update:

```sh
kubectl rollout restart deployment/verdeflow-website-depl
kubectl rollout status deployment/verdeflow-website-depl
```

The deployment uses `imagePullPolicy: Always` because `website-latest` is a mutable tag. This manifest routes both domain names to the website and redirects HTTP to HTTPS; the page canonical URL remains `https://verdeflow.org/`.
