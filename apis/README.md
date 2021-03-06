# Dragonfly Supernode API

We encourage users to experience Dragonfly in different ways. When doing this, there are three ways users could choose to interact with dragonfly mostly:

* For end-users, command line tool `dfget`, `dfdaemon` is mostly used.
* For developers, Dragonfly supernode's raw API is the original thing they would make use of. For more details about API docs, please refer to [api.md](https://alibaba.github.io/Dragonfly/api_reference/). We should also keep it in mind that doc [api.md](../docs/api/api.md) is automatically generated by [swagger2markup](https://github.com/Swagger2Markup/swagger2markup). Please **DO NOT** edit [api.md](../docs/api/api.md) directly.
* For golang developers, SDK package [client](../client) is mostly used. For more details about this SDK, please refer to [Dragonfly SDK](https://godoc.org/github.com/alibaba/Dragonfly/client).

Directory `/apis` mainly describes the second part **Dragonfly Supernode's Raw API**. If taking a look at this directory, we can find that currently it contains the following two kinds of things:

* raw API definitions via [swagger.yml](swagger.yml);
* API struct files in `/apis/types` which are used in restful API between Dragonfly Client and Server. 

## Generated API Types

Dragonfly only has generated API types via tool [swagger](https://swagger.io). These structs are all located in directory `apis/types`. For your attention, these files are totally auto generated by swagger. It is **FORBIDDEN** to edit files in directory `apis/types` directly. If you need to update one or more files in `apis/types`, please edit file `apis/swagger.yml` and auto generate corresponding API struct files. What's more, API structs in `apis/types` must be used in both client side and daemon side. If one struct defined in these files is only used in one of client and daemon side, then this struct is not proper to be defined in swagger.yml. We **MUST** move it out of `apis/types` to client side or daemon side. In a word, swagger.yml is the **ONLY** entry to define Dragonfly supernode's API. And this will guarantee the efficiency and standard of Dragonfly supernode's API defining.

## Hack Dragonfly Supernode APIs

Swagger helps us to unify the API standard, and swagger.yml is the only entry to update API. If you wish to hack Dragonfly supernode on API side, here we have several points to guide you.

### Install Go Swagger

Swagger is a simple yet powerful representation of your RESTful API. [go-swagger](https://github.com/go-swagger/go-swagger) is a tool implemented for Golang. We could install go-swagger by following official [go-swagger installation](https://github.com/go-swagger/go-swagger#binary-distribution). 

### Design API 

With swagger installed in your local $PATH, you could start to design or update API of Dragonfly supernode. Here we need to underline that every change for API should be firstly added in [swagger.yml](swagger.yml) of Drgaonfly porject.

Let's us take an example:

> add a new API "GET /version" to get the version of the supernode for client side.

To finish this, we can simplify works to two parts:

* define API path, including request method, request parameters, request content type and response status code, response body and so on;
* define API structs used by both Dragonfly supernode client and Dragonfly supernode.

For part one, we should add the following content in `paths`:

``` 
  /version:
    get:
      summary: "Get Supernode version"
      description: ""
      responses:
        200:
          schema:
            $ref: '#/definitions/SystemVersion'
          description: "no error"
        500:
          description: "server error"
          schema:
            $ref: '#/definitions/Error'
```

For part two, we need to add struct or object `SystemVersion` and `Error` in `definitions`. Added content could be seen like below:

```
  SystemVersion:
    type: "object"
    properties:
      Version:
        type: "string"
        description: "version of Dragonfly supernode"
        example: "0.1.2"
      ApiVersion:
        type: "string"
        description: "Api Version held by daemon"
        example: ""
      ......
      KernelVersion:
        type: "string"
        description: "Operating system kernel version"
        example: "3.13.0-106-generic"
      BuildTime:
        type: "string"
        description: "The time when this binary of daemon is built"
        example: "2017-08-29T17:41:57.729792388+00:00"
```

Daemon side can use `SystemVersion` easily as well. 

### Auto-generate API struct

With above two parts finished in swagger.yml, the following step is auto-generating source code for API structs. We can do that with a script [generate-swagger-models.sh](../hack/generate-swagger-models.sh). We can execute the script via:

``` shell
Dragonfly (master) $ ./hack/generate-swagger-models.sh
```

Then details you updated would auto generate source code in directory `Dragonfly/apis/types`. With the example above, you will find file `system_version.go` existing there.

### Integrate generated struct

With API structs generated, you can integrate these structs in both client side and daemon side. In client side, you could code like this integrating `SystemVersion`:

```
// SystemVersion requests daemon for system version.
func (cli *Client) SystemVersion() (*types.SystemVersion, error) {
	resp, err := cli.get("/version", nil)
	if err != nil {
		return nil, err
	}

	version := &types.SystemVersion{}
	err = decodeBody(version, resp.Body)
	ensureCloseReader(resp)

	return version, err
}
```

## Conclusion

Swagger helps Dragonfly to use a general way to design API among lots of different committers. A standard way can improve collaborating efficiency a lot.
