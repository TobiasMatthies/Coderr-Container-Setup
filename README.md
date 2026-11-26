# Coderr Container-Setup

Container setup for the **Coderr-Webapp**, built step by step in live coding sessions. The frontend and backend are included as Git submodules; the Docker configuration lives in this repository.

> **Work in progress:** This repository changes constantly during the sessions. The goal is a full Docker Compose setup. For now, only the frontend is containerized.

## Table of Contents

- [Quickstart](#quickstart)
- [Usage](#usage)
  - [Repository structure](#repository-structure)
  - [Submodules](#submodules)
  - [Frontend container](#frontend-container)
  - [Troubleshooting](#troubleshooting)

## Quickstart

Requires [Git](https://git-scm.com/downloads) and [Docker](https://docs.docker.com/get-docker/).

```bash
# 1. Clone the repository including the submodules
git clone --recurse-submodules https://github.com/TobiasMatthies/Coderr-Container-Setup
cd Coderr-Container-setup

# 2. Build the frontend image
docker build -t coderr-frontend:1.0.0 -f ./frontend/Dockerfile .

# 3. Start the container
docker run -d -p 5500:80 --name coderr-frontend coderr-frontend:1.0.0
```

Open [http://localhost:5500](http://localhost:5500).

Already cloned without `--recurse-submodules`? See [Submodules](#submodules).

## Usage

### Repository structure

```
Coderr-Container-setup/
├── .gitmodules        # Submodule definitions (path + URL)
├── Coderr-Api/        # Backend (submodule)
├── project.Coderr/    # Frontend (submodule)
├── frontend/
│   └── Dockerfile     # Nginx image that serves the frontend
└── README.md
```

| Part     | Repository                                                                          | Submodule path   |
| -------- | ----------------------------------------------------------------------------------- | ---------------- |
| Frontend | [Coderr-Frontend](https://github.com/Developer-Akademie-Backendkurs/project.Coderr) | `project.Coderr` |
| Backend  | [Coderr-Api](https://github.com/TobiasMatthies/Coderr-Api)                          | `Coderr-Api`     |

### Submodules

The frontend and backend are separate repositories. A plain `git clone` only creates empty submodule folders, so they have to be initialized.

**Initialize (after a normal clone):**

```bash
git submodule update --init --recursive
```

**Check the state:**

```bash
git submodule status
```

A leading `-` means the submodule is not initialized, a leading `+` means it is checked out at a different commit than the one recorded in this repository.

**Update to the latest remote version:**

```bash
git submodule update --remote --merge
git add Coderr-Api project.Coderr
git commit -m "Update submodules"
```

**After pulling changes of this repository** (the submodule pointers may have moved), sync your working copies:

```bash
git pull
git submodule update --init --recursive
```

### Frontend container

All commands are run from the **repository root**, because the build context must contain the `project.Coderr` folder.

The [Dockerfile](frontend/Dockerfile) uses a slim Nginx image and copies the contents of `project.Coderr/` into Nginx's web root.

| Setting         | Value                    | Where to change                      |
| --------------- | ------------------------ | ------------------------------------ |
| Base image      | `nginx:1.31.10-alpine-slim` | `frontend/Dockerfile`             |
| Container port  | `80`                     | Nginx default                        |
| Host port       | `5500`                   | `-p` flag of `docker run`            |
| Image name/tag  | `coderr-frontend:1.0.0`  | `-t` flag of `docker build`          |
| Container name  | `coderr-frontend`        | `--name` flag of `docker run`        |

Command flags explained:

- `-f ./frontend/Dockerfile` selects the Dockerfile, `.` sets the repository root as build context
- `-d` runs the container in the background
- `-p 5500:80` maps container port 80 to port 5500 on your machine

**Rebuild after changing the frontend:**

```bash
docker rm -f coderr-frontend
docker build -t coderr-frontend:1.0.0 -f ./frontend/Dockerfile .
docker run -d -p 5500:80 --name coderr-frontend coderr-frontend:1.0.0
```

**Logs, stop and clean up:**

```bash
docker logs -f coderr-frontend     # follow the logs
docker stop coderr-frontend        # stop the container
docker rm coderr-frontend          # remove the container
docker rmi coderr-frontend:1.0.0   # remove the image (optional)
```

### Troubleshooting

| Problem                                                   | Solution                                                                                 |
| --------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `project.Coderr/` or `Coderr-Api/` is empty               | Run `git submodule update --init --recursive`.                                           |
| `COPY failed: file not found ... project.Coderr`          | The submodule is not initialized, or `docker build` was not run from the repository root. |
| `port is already allocated`                               | Port 5500 is in use. Choose another host port, e.g. `-p 8080:80`.                        |
| `The container name "/coderr-frontend" is already in use` | Run `docker rm -f coderr-frontend`, then start it again.                                 |
| Changes to the frontend are not visible                   | The files are copied at build time. Rebuild the image (see above).                       |
