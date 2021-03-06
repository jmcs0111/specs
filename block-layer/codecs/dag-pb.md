# DAG-PB Spec

**Status: Descriptive - Draft**

DAG-PB does not support the full ["IPLD Data Model."](../../data-model-layer/data-model.md)

## Serial Format

The DAG-PB IPLD serial format is described with a single protobuf:

```protobuf
// An IPFS MerkleDAG Link
message PBLink {

  // binary CID (with no multibase prefix) of the target object
  optional bytes Hash = 1;

  // UTF-8 string name.
  optional string Name = 2;

  // cumulative size of target object
  optional uint64 Tsize = 3;
}

// An IPFS MerkleDAG Node
message PBNode {

  // refs to other objects
  repeated PBLink Links = 2;

  // opaque user data
  optional bytes Data = 1;
}
```

The objects link names are specified in the 'Name' field of the PBLink object.
All link names in an object must either be omitted or unique within the object.

## Logical Format

When we handle DAG-PB content at the Data Model level, we treat these objects as maps.

This layout can be expressed with [IPLD Schemas](../../schemas/README.md) as:

```ipldsch
type PBLink struct {
  Hash Bytes
  Name optional String
  Tsize optional Int
}

type PBNode struct {
  Links optional [PBLink]
  Data optional Bytes
}
```

The first node in a block of DAG-PB data will match the `PBNode` type.

When decoding data with the DAG-PB codec, maps with exactly these fields will result.

When creating data, you can create maps using the standard Data Model concepts,
and as long as they have exactly these fields, the DAG-PB codec can encode them.
If additional fields are present, the DAG-PB codec will error, because there is no way to encode them.

## Alternative Pathing

While the [logical format](#logical-format) implicitly describes a set of mechanisms for pathing over and through DAG-PB data,
DAG-PB also enjoys some other special forms of pathing in addition to the Data Model norms, and these are supported by most major applications that use DAG-PB.

This alternative pathing is covered here as part of this descriptive spec, but was developed independently of the Data Model and is thus not well standardized.
It currently differs between implementations. Please see [issue #55] for more information about the harmonization effort. This section describes the current implementations as of September 2019.

The Go and JavaScript implementation both support pathing with link names: `/<name1>/<name2>/…`.

In Go, this is the only way, which implies that is is impossible to path through nodes that don't name their links. Also neither the Data section nor the Links section/metadata are accessible through paths.

In the JavaScript implementation, there is an additional way to path through the data. It's based purely on the structure of object, i.e. `/Links/<index>/Hash/…`. This way you have direct access to the `Data`, `Links`, and `size` fields, e.g. `/Links/<index>/Hash/Data`.

These two ways of pathing can be combined, so you can access e.g. the `Data` field of a named link via `/<name/Data`. You can also use both approaches within a single path, e.g. `/<name1>/Links/0/Hash/Data` or `/Links/<index>/Hash/<name>/Data`. When using the DAG API in js-ipfs, then the pathing over the structure has precedence, so you won't be able to use named pathing on a named link called `Links`, you would need to use the index of the link instead.


## Canonical DAG-PB

Canonical DAG-PB must:

1. Contain only the specified protobuf fields.
2. Use standard protobuf encoding, with the following field orders:
  - PBNode: Links, Data
  - PBLink: Hash, Name, Tsize

Historical Note: The ordering (Links then Data) of the PBNode message is due to
a bug in the initial protobuf encoder that was used in the first implementation
of ipfs. Take care to maintain this ordering for full compatibility.

[issue #55]: https://github.com/ipld/specs/issues/55
