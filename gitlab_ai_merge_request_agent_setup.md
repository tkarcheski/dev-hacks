## **File Summary**

1. **`docker-compose.yml`**  
   Defines the Docker services for both GitLab and the AI agent.

2. **`agent/Dockerfile`**  
   Dockerfile to build the AI agent's Docker image.

3. **`agent/app.py`**  
   The main Python script containing the AI agent's logic.

4. **`agent/requirements.txt`**  
   Lists the Python dependencies for the AI agent.

5. **`agent/entrypoint.sh`**  
   Entrypoint script to initialize and run the AI agent.

---

## **Directory Structure**

```
project-root/
├── agent/
│   ├── Dockerfile
│   ├── app.py
│   ├── requirements.txt
│   └── entrypoint.sh
└── docker-compose.yml
```

---

## **1. `docker-compose.yml`**

This file sets up the Docker environment with two services: GitLab and the AI agent.

```yaml
version: '3'
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    hostname: 'gitlab'
    container_name: 'gitlab'
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - './config/gitlab:/etc/gitlab'
      - './logs/gitlab:/var/log/gitlab'
      - './data/gitlab:/var/opt/gitlab'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab'
        gitlab_rails['gitlab_shell_ssh_port'] = 22
    networks:
      - gitlab-network

  agent:
    build: ./agent
    container_name: 'gitlab-agent'
    depends_on:
      - gitlab
    environment:
      GITLAB_URL: 'http://gitlab'
      GITLAB_TOKEN: 'your_access_token'  # Replace with your token or use Docker secrets
      GITLAB_GROUP_ID: 'your_group_id'   # Replace with your GitLab group ID
    networks:
      - gitlab-network

networks:
  gitlab-network:
```

**Notes:**

- **Networks**: Both services are on the same Docker network for internal communication.
- **Environment Variables**: Replace `'your_access_token'` and `'your_group_id'` with actual values or manage them securely.

---

## **2. `agent/Dockerfile`**

This Dockerfile builds the AI agent's image.

```dockerfile
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the application code
COPY app.py .
COPY entrypoint.sh .
RUN chmod +x entrypoint.sh

# Expose port if using webhooks (optional)
# EXPOSE 5000

# Set the entrypoint
ENTRYPOINT ["./entrypoint.sh"]
```

---

## **3. `agent/requirements.txt`**

List of Python packages required for the agent.

```
python-gitlab==3.6.0
nltk==3.6.2
requests==2.25.1
pyspellchecker==0.6.2
# Uncomment if using Flask for webhooks
# flask==2.0.1
```

---

## **4. `agent/app.py`**

The main Python script containing the agent's logic.

```python
import os
import time
import gitlab
import nltk
from spellchecker import SpellChecker

# Initialize NLTK data (ensure this runs only once)
nltk.download('punkt', quiet=True)

# GitLab connection details
GITLAB_URL = os.getenv('GITLAB_URL', 'http://gitlab')
GITLAB_TOKEN = os.getenv('GITLAB_TOKEN')
GITLAB_GROUP_ID = os.getenv('GITLAB_GROUP_ID')

if not GITLAB_TOKEN or not GITLAB_GROUP_ID:
    raise EnvironmentError("GITLAB_TOKEN and GITLAB_GROUP_ID must be set as environment variables.")

# Initialize GitLab connection
gl = gitlab.Gitlab(GITLAB_URL, private_token=GITLAB_TOKEN)

def summarize_diff(merge_request):
    diffs = merge_request.diffs()
    summary = "### Summary of Changes:\n"
    for diff in diffs[:5]:  # Limit to first 5 diffs for brevity
        summary += f"- **File** `{diff['new_path']}`: Changes in lines `{diff['new_line']}`\n"
    return summary

def summarize_commits(merge_request):
    commits = merge_request.commits()
    summary = "### Commits:\n"
    for commit in commits[:5]:  # Limit to first 5 commits
        summary += f"- {commit.title}\n"
    return summary

def check_typos(text):
    spell = SpellChecker()
    words = nltk.word_tokenize(text)
    typos = spell.unknown(words)
    return typos

def check_pipeline_status(merge_request):
    pipelines = merge_request.pipelines.list()
    if pipelines:
        latest_pipeline = pipelines[0]
        return latest_pipeline.status
    else:
        return 'No pipeline found'

def check_related_issues(merge_request):
    related_issues = merge_request.related_issues()
    closes_issues = merge_request.closes_issues()
    return related_issues, closes_issues

def sync_labels(merge_request):
    project = gl.projects.get(merge_request.project_id)
    related_issues, _ = check_related_issues(merge_request)
    labels = set(merge_request.labels)
    for issue in related_issues:
        issue_detail = project.issues.get(issue['iid'])
        labels.update(issue_detail.labels)
    if labels != set(merge_request.labels):
        merge_request.labels = list(labels)
        merge_request.save()

def post_summary(merge_request, summary):
    merge_request.notes.create({'body': summary})

def process_merge_requests():
    group = gl.groups.get(GITLAB_GROUP_ID)
    merge_requests = group.mergerequests.list(state='opened', all=True)

    for mr in merge_requests:
        print(f"Processing Merge Request IID: {mr.iid}")
        # Fetch the latest version of the merge request
        mr = gl.projects.get(mr.project_id).mergerequests.get(mr.iid)

        # Skip if already processed (you may implement a flag or comment check)
        # For simplicity, we'll process all open MRs every time

        # Summarize diff and commits
        diff_summary = summarize_diff(mr)
        commits_summary = summarize_commits(mr)

        # Check for typos in the merge request title and description
        typos = check_typos((mr.title or '') + ' ' + (mr.description or ''))

        # Check pipeline status
        pipeline_status = check_pipeline_status(mr)

        # Check related or closing issues
        related_issues, closes_issues = check_related_issues(mr)

        # Sync labels
        sync_labels(mr)

        # Prepare summary
        summary = f"{diff_summary}\n{commits_summary}\n"
        summary += f"### Pipeline Status:\n- {pipeline_status}\n"

        if typos:
            summary += f"### Typos Found:\n- {', '.join(typos)}\n"

        if not related_issues and not closes_issues:
            summary += "### Notice:\n- No related or closing issues linked.\n"

        # Post the summary as a comment
        post_summary(mr, summary)
        print(f"Posted summary for Merge Request IID: {mr.iid}")

if __name__ == '__main__':
    while True:
        try:
            process_merge_requests()
        except Exception as e:
            print(f"An error occurred: {e}")
        # Wait for 5 minutes before checking again
        time.sleep(300)
```

---

## **5. `agent/entrypoint.sh`**

Entrypoint script to start the agent after ensuring GitLab is accessible.

```bash
#!/bin/bash
# entrypoint.sh

# Wait for GitLab to be ready
echo "Waiting for GitLab to become available..."
until $(curl --output /dev/null --silent --head --fail $GITLAB_URL); do
    printf '.'
    sleep 5
done

echo "GitLab is up and running."

# Start the agent
python app.py
```

**Make sure to make the script executable:**

```bash
chmod +x entrypoint.sh
```

---

## **Instructions to Run the Project**

### **Prerequisites**

- Install [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/).

### **Steps**

1. **Create Project Directory:**

   Create a directory for your project and navigate into it.

   ```bash
   mkdir gitlab-ai-agent
   cd gitlab-ai-agent
   ```

2. **Create the Directory Structure:**

   As per the directory structure mentioned above, create the `agent` directory and files.

3. **Copy the Provided Files:**

   - Create `docker-compose.yml` in the project root.
   - Inside the `agent` directory, create `Dockerfile`, `app.py`, `requirements.txt`, and `entrypoint.sh` with the contents provided.

4. **Set Environment Variables:**

   - Replace `'your_access_token'` in `docker-compose.yml` with your GitLab personal access token.
   - Replace `'your_group_id'` with the ID of your GitLab group.
   - **Alternatively**, you can use Docker environment variables or secrets to manage sensitive information.

5. **Start Docker Services:**

   ```bash
   docker-compose up -d
   ```

6. **Access GitLab Instance:**

   - Open your web browser and go to `http://localhost`.
   - Set up the root password when prompted.
   - Create a new group and note its ID (found in the URL or group settings).

7. **Generate Personal Access Token:**

   - In GitLab, go to **User Settings** > **Access Tokens**.
   - Create a token with the `api` scope.
   - Use this token in your `docker-compose.yml` or manage it securely.

8. **Test the AI Agent:**

   - Create a new project and submit a merge request.
   - The agent should process the merge request and add comments based on the rules defined.

9. **Monitor Agent Logs:**

   To view the logs and debug, run:

   ```bash
   docker-compose logs -f agent
   ```

---

## **Testing a Portion of the Project**

To start with a portion and test:

1. **Set Up Only GitLab Service:**

   - Comment out the `agent` service in `docker-compose.yml`.
   - Run `docker-compose up -d` to start GitLab.
   - Ensure GitLab is running correctly.

2. **Set Up the Agent Service Separately:**

   - Once GitLab is verified, uncomment the `agent` service.
   - Build the agent Docker image:

     ```bash
     docker-compose build agent
     ```

   - Start the agent:

     ```bash
     docker-compose up -d agent
     ```

3. **Test the Agent with a Sample Merge Request:**

   - Submit a merge request in GitLab.
   - Check if the agent processes it and adds the appropriate comments.

---

## **Next Steps and Customization**

- **Webhook Implementation (Optional):**
  - For real-time responses, implement webhooks using Flask and configure GitLab to send events to the agent.

- **Enhance the Agent Logic:**
  - Improve typo checking.
  - Add more sophisticated diff summarization.
  - Implement caching to avoid reprocessing the same merge requests.

- **Security Improvements:**
  - Use Docker secrets or environment files to manage sensitive data.
  - Limit the agent's permissions by creating a dedicated user in GitLab.

- **Logging and Monitoring:**
  - Implement logging to files or external logging services for better monitoring.

- **Testing:**
  - Write unit tests for the agent functions.
  - Use a test GitLab project to avoid affecting real data.
