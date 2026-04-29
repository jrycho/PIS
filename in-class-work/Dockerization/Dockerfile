# Declaration from where it works
FROM python:3.13-slim

WORKDIR /app

COPY ./requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Source code directory
COPY main.py main.py

# Set runtime command
# -u: unbuffered output, useful for real-time logging
CMD ["python", "-u", "main.py"]
