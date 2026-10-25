# Assignment 3 — Docker Networking

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will explore Docker networking by using default bridge, custom bridge, multiple bridge, and host network modes. You will verify container communication, service discovery, network isolation, public access, and host networking on a Linux VM or EC2 instance.

---

# Task 1 — Deploy a Standalone Application Using the Default Bridge Network

## Goal

Deploy an Nginx web server using Docker’s default bridge network and access it through the VM public IP address.

### Evidence

#### Screenshot 1 — Available Docker Networks

Add a screenshot of the terminal showing:

```bash
docker network ls
```

The output must include the default `bridge`, `host`, and `none` networks.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-1.png)

---

#### Screenshot 2 — Nginx Image Pull

Add a screenshot of the terminal showing successful completion of:

```bash
docker pull nginx:alpine
```

![ss](./screenshots/W11-SS-A3/W11-A3-SS-2.png)

---

#### Screenshot 3 — Running `myweb` Container

Add a screenshot of the terminal showing:

```bash
docker ps
```

The output must show the running `myweb` container with:

```text
0.0.0.0:80->80/tcp
```

![ss](./screenshots/W11-SS-A3/W11-A3-SS-3.png)

---

#### Screenshot 4 — Nginx Welcome Page

Add a browser screenshot showing the Nginx Welcome Page at:

```text
http://18.143.117.130/
```

Ensure that the VM public IP is visible in the address bar. Add your full name as a clear caption directly below the screenshot.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-4.png)

---

# Task 2 — Connect Containers Using a Custom Bridge Network

## Goal

Create a custom bridge network and verify that containers can communicate using container names rather than IP addresses.

### Evidence

#### Screenshot 5 — Custom Bridge Network Created

Add a screenshot of the terminal showing `mynetwork` in:

```bash
docker network ls
```

![ss](./screenshots/W11-SS-A3/W11-A3-SS-5.png)

---

#### Screenshot 6 — Running `web` and `client` Containers

Add a screenshot of the terminal showing:

```bash
docker ps
```

The output must show both `web` and `client` containers running without published host ports.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-6.png)

---

#### Screenshot 7 — Service Discovery by Container Name

Add a screenshot of the terminal showing successful output from:

```bash
docker exec client wget -qO- http://web
```

The output must display the Nginx Welcome Page HTML.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-7.png)

---

#### Screenshot 8 — Custom Network Inspection

Add a screenshot of the terminal showing:

```bash
docker network inspect mynetwork
```

The output must show both `web` and `client` connected to `mynetwork`.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-8.png)

---

# Task 3 — Demonstrate Multi-Network Isolation

## Goal

Deploy frontend, backend, and database containers across two separate Docker networks. Verify allowed communication and confirm that the frontend cannot directly reach the database.

### Evidence

#### Screenshot 9 — Two Docker Networks

Add a screenshot of the terminal showing:

```bash
docker network ls
```

The output must include both `frontend-network` and `backend-network`.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-9.png)

---

#### Screenshot 10 — Running Multi-Network Containers

Add a screenshot of the terminal showing:

```bash
docker ps
```

The output must show:

- `frontend` with the published port mapping `0.0.0.0:80->80/tcp`
- `backend` without a published host port
- `db` without a published host port

![ss](./screenshots/W11-SS-A3/W11-A3-SS-10.png)

---

#### Screenshot 11 — Frontend Network Inspection

Add a screenshot of the terminal showing:

```bash
docker network inspect frontend-network
```

The output must show `frontend` and `backend`.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-11.png)

---

#### Screenshot 12 — Backend Network Inspection

Add a screenshot of the terminal showing:

```bash
docker network inspect backend-network
```

The output must show `backend` and `db`.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-12.png)

---

#### Screenshot 13 — Frontend-to-Backend Communication

Add a screenshot of the terminal showing successful output from:

```bash
docker exec frontend wget -qO- http://backend
```

The output must display the Nginx Welcome Page HTML.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-13.png)

---

#### Screenshot 14 — Backend-to-Database Communication

Add a screenshot of the terminal showing a successful connection to `db` on port `27017` from the `backend` container.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-14.png)

---

#### Screenshot 15 — Frontend-to-Database Isolation

Add a screenshot of the terminal showing that the `frontend` container cannot reach `db` on port `27017`.

The output must include:

```text
Expected result: frontend cannot reach db
```

![ss](./screenshots/W11-SS-A3/W11-A3-SS-15.png).

---

#### Screenshot 16 — Public Frontend Access

Add a browser screenshot showing the Nginx Welcome Page from the `frontend` container at:

```text
http://<YOUR-VM-PUBLIC-IP>
```

Ensure that the VM public IP is visible in the address bar. Add your full name as a clear caption directly below the screenshot.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-16.png).

---

# Task 4 — Deploy an Application Using Docker Host Network Mode

## Goal

Run an Nginx container using Docker host network mode and compare it with bridge networking.

### Evidence

#### Screenshot 17 — Running Host-Networked Container

Add a screenshot of the terminal showing:

```bash
docker ps
```

The output must show the running `fastapp` container.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-17.png).

---

#### Screenshot 18 — Host Network Mode Verification

Add a screenshot of the terminal showing output from:

```bash
docker inspect fastapp | grep '"NetworkMode"'
```

The output must confirm:

```text
"NetworkMode": "host"
```

![ss](./screenshots/W11-SS-A3/W11-A3-SS-18.png).

---

#### Screenshot 19 — Host-Networked Nginx Page

Add a browser screenshot showing the Nginx Welcome Page at:

```text
http://<YOUR-VM-PUBLIC-IP>
```

Ensure that the VM public IP is visible in the address bar. Add your full name as a clear caption directly below the screenshot.

![ss](./screenshots/W11-SS-A3/W11-A3-SS-19.png).

---

#### Screenshot 20 — Host-Networked Container Cleanup

Add a screenshot of the terminal showing successful completion of:

```bash
docker stop fastapp
docker rm fastapp
```

![ss](./screenshots/W11-SS-A3/W11-A3-SS-20.png).

---

# Networking Notes

Write a short note explaining:

- Default bridge networking
- Container-name communication on a custom bridge network
- Why the frontend could not access the database in Task 3
- The difference between bridge mode and host network mode

- Default bridge networking**
When a container is started without `--network`, Docker attaches it to the default `bridge` network. The container gets a private IP and can reach the internet, but it is not reachable from outside unless a port is published. In Task 1, `-p 80:80` mapped the EC2 instance's port 80 to port 80 in the `myweb` container, which made the Nginx page available at the instance's public IP. Containers on the default bridge cannot resolve each other by name.

- Container-name communication on a custom bridge network**
A user-defined bridge network such as `mynetwork` includes Docker's built-in DNS. In Task 2, `client` reached `web` with `wget -qO- http://web`, and Docker resolved the name `web` to its IP (`172.18.0.2`). Neither container published a port, so they could reach each other but could not be reached from outside the instance. Using names avoids hardcoding container IPs, which can change when a container is recreated.

- Why the frontend could not access the database in Task 3**
`frontend` was attached only to `frontend-network`, and `db` only to `backend-network`. The two containers shared no network, so the name `db` did not resolve from `frontend` (`nc: bad address 'db'`). `backend` was connected to both networks, with a separate IP on each (`172.19.0.2` and `172.20.0.3`), which made it the only path between the tiers. Only `frontend` published port 80, so `backend` and `db` stayed internal.

- Difference between bridge mode and host network mode**
In bridge mode, each container has its own isolated network namespace and private IP, and outside access requires a port mapping (`-p`). In host mode (`--network host`), the container shares the host's network stack directly. In Task 4, `fastapp` had no port mapping, `docker ps` showed an empty PORTS column, and Nginx was still reachable on the instance's port 80. Host mode removes the NAT layer, but it also removes network isolation, and the host port must be free before the container starts.

---

# LinkedIn Requirement

## Goal

Create a LinkedIn post about the Docker networking modes explored, one key lesson about container isolation, and the learning outcomes from this assignment.

### Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`Add your URL here`

---

#### LinkedIn Post Screenshot

Add a screenshot of the published LinkedIn post here.

---

# Submission Instructions

- Complete all tasks in sequence.
- Include Screenshots 1–20 exactly as specified.
- Include the Networking Notes section.
- Include the LinkedIn post URL and screenshot.
- Ensure that your full name is visible in all terminal screenshots.
- Add your full name as a caption below each browser screenshot that shows the standard Nginx page.
- Do not expose private keys, passwords, access keys, tokens, account IDs, or other sensitive information.

---

# Completion Checklist

- [✓] Completed on a Linux VM or EC2 instance
- [✓] Docker Engine is running
- [✓] HTTP port 80 is allowed in the VM firewall or cloud security rules
- [✓] Default bridge networking verified
- [✓] Custom bridge network created
- [✓] Container-name communication verified
- [✓] `frontend-network` and `backend-network` created
- [✓] Frontend-to-backend communication verified
- [✓] Backend-to-database communication verified
- [✓] Frontend-to-database isolation verified
- [✓] Only the frontend published port 80 in Task 3
- [✓] Host network mode verified
- [✓] All required screenshots included
- [✓] Networking Notes completed
- [ ] LinkedIn post URL and screenshot included
- [✓] Full name visible in terminal screenshots
- [✓] Browser screenshots include full-name captions
- [✓] No sensitive information exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*
