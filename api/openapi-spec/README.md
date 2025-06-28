# Kubernetes's OpenAPI Specification

This folder contains an [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification) for Kubernetes API.

## Vendor Extensions

Kubernetes extends OpenAPI using these extensions. Note the version that
extensions have been added.

### `x-kubernetes-group-version-kind`

Operations and Definitions may have `x-kubernetes-group-version-kind` if they
are associated with a [kubernetes resource](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources).


For example:

``` json
"paths": {
    ...
    "/api/v1/namespaces/{namespace}/pods/{name}": {
        ...
        "get": {
        ...
            "x-kubernetes-group-version-kind": {
            "group": "",
            "version": "v1",
            "kind": "Pod"
            }
        }
    }
}
```

### `x-kubernetes-action`

Operations and Definitions may have `x-kubernetes-action` if they
are associated with a [kubernetes resource](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources).
Action can be one of `get`, `list`, `put`, `patch`, `post`, `delete`, `deletecollection`, `watch`, `watchlist`, `proxy`, or `connect`.


For example:

``` json
"paths": {
    ...
    "/api/v1/namespaces/{namespace}/pods/{name}": {
        ...
        "get": {
        ...
            "x-kubernetes-action": "list"
        }
    }
}
```

### `x-kubernetes-patch-strategy` and `x-kubernetes-patch-merge-key`

Some of the definitions may have these extensions. For more information about PatchStrategy and PatchMergeKey see
[strategic-merge-patch](https://git.k8s.io/community/contributors/devel/sig-api-machinery/strategic-merge-patch.md).

### `x-kubernetes-list-type`

Arrays may have `x-kubernetes-list-type` to further describe their topology.
This extension must only be used on lists and may have 3 possible values:

1) `atomic`: the list is treated as a single entity, like a scalar.
     Atomic lists will be entirely replaced when updated. This extension
     may be used on any type of list (struct, scalar, ...).
2) `set`:
     Sets are lists that must not have multiple items with the same value. Each
     value must be a scalar, an object with x-kubernetes-map-type `atomic` or an
     array with x-kubernetes-list-type `atomic`.
3) `map`:
     These lists are like maps in that their elements have a non-index key
     used to identify them. Order is preserved upon merge. The map tag
     must only be used on a list with elements of type object.

Defaults to atomic for arrays.

For example:

``` json
"spec": {
    "type": "object",
    "properties": {
        "containers": {
            "type": "array",
            "x-kubernetes-list-type": "atomic",
            "items": {
                "type": "object"
            }
        }
    }
}
```

### `x-kubernetes-list-map-keys`

Arrays with `x-kubernetes-list-type` set to `map` may have `x-kubernetes-list-map-keys` 
to specify the keys used as the index of the map.

This tag MUST only be used on lists that have the "x-kubernetes-list-type"
extension set to "map". Also, the values specified for this attribute must
be a scalar typed field of the child structure (no nesting is supported).

The properties specified must either be required or have a default value,
to ensure those properties are present for all list items.

For example:

``` json
"spec": {
    "type": "object", 
    "properties": {
        "containers": {
            "type": "array",
            "x-kubernetes-list-type": "map",
            "x-kubernetes-list-map-keys": ["name"],
            "items": {
                "type": "object",
                "properties": {
                    "name": {
                        "type": "string"
                    }
                },
                "required": ["name"]
            }
        }
    }
}
```

### `x-kubernetes-map-type`

Objects may have `x-kubernetes-map-type` to further describe their topology.
This extension must only be used when type is object and may have 2 possible values:

1) `granular`:
     These maps are actual maps (key-value pairs) and each fields are independent
     from each other (they can each be manipulated by separate actors). This is
     the default behaviour for all maps.
2) `atomic`: the list is treated as a single entity, like a scalar.
     Atomic maps will be entirely replaced when updated.

For example:

``` json
"spec": {
    "type": "object",
    "properties": {
        "nodeSelector": {
            "type": "object",
            "x-kubernetes-map-type": "atomic"
        }
    }
}
```

### `x-kubernetes-preserve-unknown-fields`

Objects may have `x-kubernetes-preserve-unknown-fields` to stop the API server
decoding step from pruning fields which are not specified in the validation schema.
This affects fields recursively, but switches back to normal pruning behaviour if nested
properties or additionalProperties are specified in the schema.
This can either be true or undefined. False is forbidden.

For example:

``` json
"spec": {
    "type": "object",
    "x-kubernetes-preserve-unknown-fields": true,
    "properties": {
        "knownField": {
            "type": "string"
        }
    }
}
```

### `x-kubernetes-embedded-resource`

Objects may have `x-kubernetes-embedded-resource` to define that the value is an
embedded Kubernetes runtime.Object, with TypeMeta and ObjectMeta. The type must be object.
It is allowed to further restrict the embedded object. kind, apiVersion and metadata
are validated automatically. x-kubernetes-preserve-unknown-fields is allowed to be true,
but does not have to be if the object is fully specified (up to kind, apiVersion, metadata).

For example:

``` json
"spec": {
    "type": "object",
    "properties": {
        "embeddedObject": {
            "type": "object",
            "x-kubernetes-embedded-resource": true
        }
    }
}
```

### `x-kubernetes-int-or-string`

Fields may have `x-kubernetes-int-or-string` to specify that this value is
either an integer or a string. If this is true, an empty type is allowed and type
as child of anyOf is permitted if following one of the following patterns:

1) anyOf:
   - type: integer
   - type: string
2) allOf:
   - anyOf:
     - type: integer
     - type: string
   - ... zero or more

For example:

``` json
"spec": {
    "type": "object",
    "properties": {
        "port": {
            "x-kubernetes-int-or-string": true,
            "anyOf": [
                {"type": "integer"},
                {"type": "string"}
            ]
        }
    }
}
```

### `x-kubernetes-validations`

Objects and their properties may have `x-kubernetes-validations` to describe a list of 
validation rules written in the CEL expression language. Each validation rule is scoped to 
the location of the x-kubernetes-validations extension in the schema and has access to the 
object being validated via the `self` variable.

For example:

``` json
"spec": {
    "type": "object",
    "properties": {
        "replicas": {
            "type": "integer",
            "minimum": 1
        },
        "maxReplicas": {
            "type": "integer"
        }
    },
    "x-kubernetes-validations": [
        {
            "rule": "self.maxReplicas >= self.replicas",
            "message": "maxReplicas must be greater than or equal to replicas"
        }
    ]
}
```
