# OpenFaas Python3 FastAPI using conda Template

A Python [FastAPI](https://github.com/tiangolo/fastapi) template that make use of the incubator project [of-watchdog](https://github.com/openfaas-incubator/of-watchdog) relying on a [conda](http:conda.io) environment.yml file for the Python dependencies.

Templates available in this repository:
- python3-fastapi-conda

This template is heavily based on https://github.com/loudsquelch/openfaas-python3-fastapi-template but changes the way Python dependencies are managed (does not use `pip` and `requirements.txt` files)  

Rational: this template targets the development of openfaas functions using Python that require conda packages to run.

## Downloading the templates
```
$ faas template pull https://github.com/terradue/openfaas-python3-fastapi-conda-template
```

## Using the python3-fastapi-conda templates

Create a new function

```
$ faas new --lang python3-fastapi-conda <fn-name>
```

Update the `environment.yml` file with additional conda `dependencies` you may need to implement your function. 

**Don't change the environment name**

**Don't remove the `fastapi` and `uvicorn` dependencies, those are mandatory.

```yaml
name: env_openfaas
channels:
  - conda-forge
dependencies:
  - python
  - fastapi
  - uvicorn
```

Build, push, and deploy

```
$ faas up -f <fn-name>.yml
```

Set your OpenFaaS gateway URL. For example:

```
$ OPENFAAS_URL=http://127.0.0.1:8080
```

Test the new function

```
$ curl -i $OPENFAAS_URL/function/<fn-name>
```

## Usage

### Request Body

The function handler is passed one argument - *req* which contains the request body.

### Response Bodies

By default, the template will automatically attempt to set the correct Content-Type header for you based on the type of response. 