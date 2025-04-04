FROM python:3-slim

RUN apt-get update && apt-get install -y \
    python3-dev \
    nodejs \
    npm

RUN rm -rf /var/lib/apt/lists/*

RUN pip install uv

RUN npm install -g supergateway

EXPOSE 8000

ENTRYPOINT ["supergateway"]

CMD ["--help"]
