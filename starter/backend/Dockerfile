FROM public.ecr.aws/docker/library/python:3.10-slim-bookworm

ENV FLASK_RUN_HOST=0.0.0.0
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        gcc \
        libpcre3 \
        libpcre3-dev \
        libffi-dev \
        libssl-dev && \
    rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir pipenv

RUN mkdir -p /var/www/app /app

COPY Pipfile Pipfile.lock /app/

RUN cd /app && pipenv install --system --deploy

COPY . /app

EXPOSE 5000

CMD ["uwsgi", "--http", ":5000", "--master", "--enable-threads", "-s", "/var/www/app/app.sock", "--manage-script-name", "--mount", "/=app:app"]