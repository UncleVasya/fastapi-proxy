## - How to run tests?

Cloning repository.
```
git clone https://github.com/cedvict/fastapi-proxy.git
cd fastapi-proxy
cd tests
```

To begin with, we raise our test microservice and the gateway.
```
uvicorn "fastapi_gateway_service.main:app" --host "localhost" --port 8001
uvicorn "fastapi_microservice.main:app" --host "localhost" --port 8002
```

And run tests.
```
pythom -m pytest tests
```
