FROM python:slim

# Set environment variables for non-interactive installations
ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /app

ARG apt_cacher
RUN if [ -n "$apt_cacher" ] ; then \
    echo "Acquire::http { Proxy \"http://${apt_cacher}:3142\"; };" >> /etc/apt/apt.conf.d/01proxy ; \
    fi

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt \
    && mkdir /data

COPY . .

# Set up entrypoint
RUN chmod +x entrypoint.sh
ENTRYPOINT ["./entrypoint.sh"]
