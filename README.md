<!---
{
  "id": "ae09827a-4af8-4d4f-bf38-582731a2f110",
  "teaches": "Running a FastAPI Service with `systemd` and Testing with `curl`",
  "depends_on": [
    "a4b0b901-5c73-48b4-9eb8-5cedc4d8c67b",
    "c1f2cd2b-3ffc-44a8-86b1-111f9d246c10",
    "e8add8e9-7a67-4b50-af89-6c1ce6558e0d"
  ],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-06",
  "keywords": ["systemd", "fastapi", "curl", "python", "web services"]
}
--->

# Running a FastAPI Service with `systemd` and Testing with `curl`

> In this exercise you will learn how to deploy a FastAPI application as a systemd service. Furthermore we will explore how to interact with this API using `curl`.

### Introduction

FastAPI is a modern, fast (high-performance) web framework for building APIs with Python 3.6+ based on standard Python type hints. It is designed to be easy to use while enabling the development of robust, production-grade web services. One powerful way to manage and run such services on Linux systems is with `systemd`, which provides tools to supervise and control services with features like automatic restarts, logging integration, and startup configuration.

In this exercise, you will:

* Create a simple FastAPI application with two endpoints.
* Run the application using `uvicorn`, the ASGI server recommended for FastAPI.
* Set up a systemd user-level service to run the app persistently.
* Test the deployed endpoints using the `curl` command-line HTTP client.

This exercise will help you understand how to turn Python-based APIs into background services that are boot-persistent and easy to monitor, control, and test.

### Further Readings and Other Sources

* [FastAPI Documentation](https://fastapi.tiangolo.com/)
* [Python HTTP Clients: curl and Requests](https://requests.readthedocs.io/)
* [Red Hat Developer Blog: Run your Python application as a systemd service](https://developers.redhat.com/blog/2021/09/21/how-to-run-your-python-applications-as-a-systemd-service)
* [systemd man pages](https://www.freedesktop.org/software/systemd/man/systemd.html)

---

### Tasks

#### 1. Set Up the FastAPI Project

Begin by creating a directory to hold your FastAPI project:

```sh
mkdir ~/fastapi_service
cd ~/fastapi_service
```

Create a Python virtual environment to keep your dependencies isolated:

```sh
python3 -m venv venv
source venv/bin/activate
```

Install the necessary packages — FastAPI itself and `uvicorn`, the server:

```sh
pip install fastapi uvicorn
```

Now, create the main application file:

```sh
nano main.py
```

Insert the following code:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello, World!"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

This defines two simple endpoints: one root path that returns a welcome message, and one parameterized path for item lookup.

#### 2. Create the systemd Service File

To make this FastAPI app run as a background service, we need to create a `systemd` unit file. First, find the absolute path to your virtual environment’s `uvicorn` binary:

```sh
which uvicorn
```

Copy the path output — it will look like `/home/yourusername/fastapi_service/venv/bin/uvicorn`.

Create the user systemd directory if it doesn't exist:

```sh
mkdir -p ~/.config/systemd/user
```

Create the unit file:

```sh
nano ~/.config/systemd/user/fastapi.service
```

Paste and adjust the following content:

```ini
[Unit]
Description=FastAPI Service
After=network.target

[Service]
WorkingDirectory=/home/YOUR_USERNAME/fastapi_service
ExecStart=/home/YOUR_USERNAME/fastapi_service/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=default.target
```

Replace `YOUR_USERNAME` with your actual Linux username.

#### 3. Enable and Start the Service

Reload the systemd manager to recognize your new unit:

```sh
systemctl --user daemon-reload
```

Enable the service so it starts automatically on login:

```sh
systemctl --user enable fastapi.service
```

Start the service:

```sh
systemctl --user start fastapi.service
```

Check the status:

```sh
systemctl --user status fastapi.service
```

Look for lines indicating that the service is active and listening on port 8000.

#### 4. Test the API with `curl`

You can now interact with your FastAPI service using `curl`, which sends HTTP requests directly from your terminal.

To test the root endpoint:

```sh
curl http://localhost:8000/
```

Expected output:

```json
{"message":"Hello, World!"}
```

Test an item endpoint without a query string:

```sh
curl http://localhost:8000/items/42
```

Expected:

```json
{"item_id":42,"q":null}
```

Test with a query string:

```sh
curl "http://localhost:8000/items/42?q=test"
```

Expected:

```json
{"item_id":42,"q":"test"}
```

---

### Questions

1. What does the `Restart=always` directive ensure in your systemd service?
2. How is a user-level service different from a system-level service in `systemd`?
3. Why should you always use absolute paths in systemd unit files?
4. In what ways is `curl` useful for testing RESTful APIs?

---

### Advice

Make sure to use your own home directory path when defining the `WorkingDirectory` and `ExecStart`. Forgetting this is a common mistake and leads to failed service starts. It's also a good practice to run your FastAPI application manually with `uvicorn main:app` before turning it into a service — this ensures it runs as expected. Finally, for professional deployment, consider using a reverse proxy like Nginx in front of FastAPI, and possibly run it under `gunicorn` with multiple workers for scalability. This setup is ideal for personal or development use, and builds essential skills for real-world deployments.
