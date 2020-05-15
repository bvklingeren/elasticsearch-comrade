FROM python:3.8.3-slim@sha256:836f9dd208ce67d0b675ea1a05d1ff05c65449c0e56071cab67b9e5b03dc2557
WORKDIR /app
COPY setup.py MANIFEST.in server/Pipfile* ./
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc python-dev g++ make libffi-dev libssl-dev &&\
    mkdir /app/comrade && \
    pip install pipenv && \
    pipenv install -e . --skip-lock && \
    apt-get purge -y --auto-remove gcc python-dev g++ make libffi-dev libssl-dev && \
    find /usr/local/lib/python3.7/ -type d -name __pycache__ | xargs rm -rf && \
    find /root/.local/ -type d -name __pycache__ | xargs rm -rf && \
    rm -rf ~/.cache /var/cache && \
    rm -rf /var/lib/apt/lists/*
COPY server/ /app/comrade/
EXPOSE 8000
CMD pipenv run comrade
