# Building and Copying frontend resources
FROM node:12-slim as builder

# Copy only specific folders so the entire frontend doesn't need to be recompiled
# if an adjustment is made to the python config
COPY static /usr/local/amundsen/frontend/configs/static
COPY upstream /usr/local/amundsen/frontend/upstream
COPY static_build.sh /usr/local/amundsen/frontend/
WORKDIR /usr/local/amundsen/frontend/

RUN chmod 755 static_build.sh && \
    ./static_build.sh

FROM python:3.7-slim

ENV LANG="en_US.utf8" \
    AMUNDSEN_SERVICE_NAME="frontend" \
    # Use sock
    GUNICORN_BIND="0.0.0.0:5000" \
    GUNICORN_TIMEOUT=60 \
    GUNICORN_WORKERS=2 \
    PYTHONPATH=$PYTHONPATH:/usr/local/amundsen/frontend \
    FLASK_DEBUG=0 \
    FLASK_ENV=production

ENV GUNICORN_CMD_ARGS="--bind=${GUNICORN_BIND} --timeout=${GUNICORN_TIMEOUT} --workers=${GUNICORN_WORKERS} --access-logfile - -"

WORKDIR /usr/local/amundsen/frontend/

# Serve the static contents from the custom builder
COPY --from=builder /usr/local/amundsen/frontend/configs/ configs/

# Copy only what's necessary for the install first
COPY upstream /usr/local/amundsen/frontend/upstream
COPY requirements.txt /usr/local/amundsen/frontend/

# Install the local copy of upstream
RUN pip install upstream/ && \
    # Install the custom requirements
    pip install -r requirements.txt

# Copy the other commonly edited files
COPY configs configs/
COPY wsgi.py .

EXPOSE 5000

CMD ["gunicorn", "wsgi:app"," --timeout 1800", "--preload"]
