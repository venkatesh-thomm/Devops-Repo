
FROM python:3.11-slim AS builder

WORKDIR /app 

COPY requirements.txt . 


RUN pip install --no-cache --prefix=/install -r requirements.txt 


FROM python:3.10 

WORKDIR /app

copy --from=builder  /install 