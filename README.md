# Fastapi-proxy is async single entry point for microservices 
(This is a fork [fastapi-gateway](https://github.com/dotX12/fastapi-gateway)).

API Gateway performs many tasks:
- accepts, processes and distributes requests
- controls traffic
- monitors and controls access and security
- caching
- throttling.

Initially, this project was created for myself, I needed to implement identification, authentication and authorization.

In the future, there was a need to limit requests for each user on every endpoint, create API plans.

There were a lot of microservices and to keep in each microservice the logic for limiting endpoints, security logic, logging etc. - meaningless.

Therefore, all this functionality is located at a single entry point, which already implements all the necessary tasks with security, limiting, etc., while microservices now directly solve their tasks.

## Installation

```
pip install fastapi_proxy
```

## Example


<code>Example of use (long code)</code>


```python3
from starlette import status
from starlette.requests import Request
from starlette.responses import Response
from fastapi_proxy import route
from fastapi import FastAPI
from pydantic import BaseModel
from fastapi import Depends
from fastapi.security import APIKeyHeader
from starlette import status
from starlette.exceptions import HTTPException

app = FastAPI(title='API Gateway')
SERVICE_URL = "http://microservice.localtest.me:8002"

API_KEY_NAME = "x-api-key"

api_key_header = APIKeyHeader(
    name=API_KEY_NAME,
    auto_error=False
)


def check_api_key(key: str = Depends(api_key_header)):
    if key:
        return key
    raise HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="You didn't pass the api key in the header! Header: x-api-key",
    )


class FooModel(BaseModel):
    example_int: int
    example_str: str


@route(
    request_method=app.post,
    service_url=SERVICE_URL,
    gateway_path='/query_and_body_path/{path}',
    service_path='/v1/query_and_body_path/{path}',
    query_params=['query_int', 'query_str'],
    body_params=['test_body'],
    status_code=status.HTTP_200_OK,
    tags=['Query', 'Body', 'Path'],
    dependencies=[
        Depends(check_api_key)
    ],
)
async def check_query_params_and_body(
        path: int, query_int: int, query_str: str,
        test_body: FooModel, request: Request, response: Response
):
    pass
  ```


 ## How to use?

- **request_method** -  is a callable (like app.get, app.post, foo_router.patch and so on.).  
- **service_url** - the path to the endpoint on another service (like "https://microservice1.example.com").  
- **service_path** - the path to the method in microservice (like "/v1/microservice/users").  
- **proxy_path** - is the path to bind proxy.
- **override_headers** - Boolean value allows you to return all the headlines that were created by microservice in proxy.  
- **query_params** - used to extract query parameters from endpoint and transmission to microservice
- **form_params** -  used to extract form model parameters from endpoint and transmission to microservice
- **param body_params** - used to extract body model from endpoint and transmission to microservice

For example, your proxy api is located here - *https://proxy.example.com* and the path to endpoint (**proxy_path**) - "/users" then the full way to this method will be - *https://proxy.example.com/users*

⚠ **Be sure to transfer the name of the argument to the router, which is in the endpoint func!**  

```
query_params - List[Query]
body_params - List[Body]
form_params - List[File, Form]
 ```


<img alt="" width="450" height="456" src="https://user-images.githubusercontent.com/64792903/130335866-82be1684-cd54-43d3-8e0e-4013176a352a.jpg">

## Build and publish

Refer to [https://packaging.python.org/en/latest/tutorials/packaging-projects/](https://packaging.python.org/en/latest/tutorials/packaging-projects/)

Step:
- python -m build
- python -m twine upload --repository pypi dist/*