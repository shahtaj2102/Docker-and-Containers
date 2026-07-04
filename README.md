# Docker & Containers

Took an app from "runs on my machine" to a reproducible, networked, multi-container setup: wrote the Dockerfile, orchestrated it with Compose alongside a database, persisted its data, and pushed the built image to a private registry I'd already stood up myself.

**Note on context:** This is hands-on lab work from the TechWorld with Nana DevOps bootcamp (Module 7), building directly on the DigitalOcean and Nexus work in my other two repos. The starting app (`starting-code/app`) is course-provided scaffolding, I'm disclosing that rather than letting it look like more than it is. `final-code/` is where my actual Dockerfile, Compose file, and configuration work live.

## Why This Exists

Containerizing an app is the first real step toward CI/CD, you can't build a Jenkins pipeline or deploy to Kubernetes around something that isn't packaged consistently yet. This project was about getting that packaging step right: a proper Dockerfile, a private place to store the image, and a way to run the whole stack (app, database, admin UI) with one command instead of a page of manual steps.
