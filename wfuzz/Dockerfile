FROM python:3.6-alpine

RUN apk add --update --no-cache git openssl-dev musl-dev curl-dev gcc

ENV PYCURL_SSL_LIBRARY=openssl

RUN git clone https://github.com/xmendez/wfuzz.git

WORKDIR /wfuzz

RUN pip install -r requirements.txt

RUN apk del openssl-dev musl-dev gcc

ENTRYPOINT ["python", "src/wfuzz-cli.py"]
