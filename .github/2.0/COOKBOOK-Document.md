# Temporary Cookbook on `Document`/`DocumentArray` 2.0 API

`Document` is the basic data type that Jina operates with. Text, picture, video, audio, image, 3D mesh, they are
all `Document` in Jina.

`DocumentArray` is a sequence container of `Document`. It is the first-class citizen of `Executor`, serving as the input
& output.

One can say `Document` to Jina is like `np.float` to Numpy, then `DocumentArray` is like `np.ndarray`.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [`Document` API](#document-api)
  - [`Document` Attributes](#document-attributes)
    - [Meta Attributes](#meta-attributes)
    - [Content Attributes](#content-attributes)
    - [Recursive Attributes](#recursive-attributes)
    - [Relevance Attributes](#relevance-attributes)
  - [Construct a `Document`](#construct-a-document)
    - [Construct with Multiple Attributes](#construct-with-multiple-attributes)
    - [Construct from Dict or JSON String](#construct-from-dict-or-json-string)
    - [Construct from Another `Document`](#construct-from-another-document)
- [`DocumentArray` API](#documentarray-api)
- [Extracting Multiple Attributes](#extracting-multiple-attributes)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## `Document` API

### `Document` Attributes

A `Document` object has the following attributes, which can be put into the following categories:

| | | 
|---|---|
| Meta attributes | `.id`, `.weight`, `.uri`, `.mime_type`, `.location`, `.offset`, `.modality` |
| Content attributes | `.buffer`, `.blob`, `.text`, `.content`, `.embedding`, `.tags` | 
| Recursive attributes | `.chunks`, `.matches`, `.granularity`, `.adjacency` |
| Relevance attributes | `.score`, `.evaluations` |

#### Recursive Attributes

`Document` can be recurred in both horizontal & vertical way.

|     |     |
| --- | --- |
| `doc.chunks` | The list of sub-documents of this document. They have `granularity + 1` but same `adjacency` |
| `doc.matches` | The list of matched documents of this document. They have `adjacency + 1` but same `granularity` |
|  `doc.granularity` | The recursion "depth" of the recursive chunks structure |
|  `doc.adjacency` | The recursion "width" of the recursive match structure |

#### Relevance Attributes

|     |     |
| --- | --- |
| `doc.score` | The relevance information of this document |
| `doc.evaluations` | The evaluation information of this document |

### Construct a `Document`

##### Content Attributes

|     |     |
| --- | --- |
| `doc.buffer` | The raw binary content of this document |
| `doc.blob` | The `ndarray` of the image/audio/video document |
| `doc.text` | The text info of the document |
| `doc.content` | One of the above non-empty field |
| `doc.embedding` | The embedding `ndarray` of this Document |
| `doc.tags` | A structured data value, consisting of field which map to dynamically typed values |

You can assign `str`, `ndarray`, `buffer` to a `Document`.

```python
from jina import Document
import numpy as np

d1 = Document(content='hello')
d2 = Document(content=b'\f1')
d3 = Document(content=np.array([1, 2, 3]))
```

```text
<jina.types.document.Document id=2ca74b98-aed9-11eb-b791-1e008a366d48 mimeType=text/plain text=hello at 6247702096>
<jina.types.document.Document id=2ca74f1c-aed9-11eb-b791-1e008a366d48 buffer=DDE= mimeType=text/plain at 6247702160>
<jina.types.document.Document id=2caab594-aed9-11eb-b791-1e008a366d48 blob={'dense': {'buffer': 'AQAAAAAAAAACAAAAAAAAAAMAAAAAAAAA', 'shape': [3], 'dtype': '<i8'}} at 6247702416>
```

The content will be automatically assigned to one of `text`, `buffer`, `blob` fields, `id` and `mime_type` are
auto-generated when not given.

#### Construct with Multiple Attributes

##### Meta Attributes

|     |     |
| --- | --- |
| `doc.id` | A hexdigest that represents a unique document ID |
| `doc.weight` | The weight of this document |
| `doc.uri` | A uri of the document could be: a local file path, a remote url starts with http or https or data URI scheme |  
| `doc.mime_type` | The mime type of this document |
| `doc.location` | The position of the doc, could be start and end index of a string; could be x,y (top, left) coordinate of an image crop; could be timestamp of an audio clip |
| `doc.offset` | The offset of this doc in the previous granularity document|
| `doc.modality` | An identifier to the modality this document belongs to|

You can assign multiple attributes in the constructor via:

```python
from jina import Document

d = Document(content='hello',
             uri='https://jina.ai',
             mime_type='text/plain',
             granularity=1,
             adjacency=3,
             tags={'foo': 'bar'})
```

```text
<jina.types.document.Document id=e01a53bc-aedb-11eb-88e6-1e008a366d48 uri=https://jina.ai mimeType=text/plain tags={'foo': 'bar'} text=hello granularity=1 adjacency=3 at 6317309200>
```

#### Construct from Dict or JSON String

You can build a `Document` from `dict` or a JSON string.

```python
from jina import Document
import json

d = {'id': 'hello123', 'content': 'world'}
d1 = Document(d)

d = json.dumps({'id': 'hello123', 'content': 'world'})
d2 = Document(d)
```

##### Parsing Unrecognized Fields

Unrecognized fields in Dict/JSON string are automatically put into `.tags` field.

```python
from jina import Document

d1 = Document({'id': 'hello123', 'foo': 'bar'})
```

```text
<jina.types.document.Document id=hello123 tags={'foo': 'bar'} at 6320791056>
```

You can use `field_resolver` to map the external field name to `Document` attributes, e.g.

```python
from jina import Document

d1 = Document({'id': 'hello123', 'foo': 'bar'}, field_resolver={'foo': 'content'})
```

```text
<jina.types.document.Document id=hello123 mimeType=text/plain text=bar at 6246985488>
```

#### Construct from Another `Document`

Assigning a `Document` object to another `Document` object will make a shallow copy.

```python
from jina import Document

d = Document(content='hello, world!')
d1 = d

assert id(d) == id(d1)  # True
```

To make a deep copy, use `copy=True`,

```python
d1 = Document(d, copy=True)

assert id(d) == id(d1)  # False
```

### Serialize `Document`

You can serialize a `Document` into JSON string or Python dict or binary string via

```python
from jina import Document
d = Document(content='hello, world')
d.json()
```

```
{
  "id": "6a1c7f34-aef7-11eb-b075-1e008a366d48",
  "mimeType": "text/plain",
  "text": "hello world"
}
```

```python
d.dict()
```

```
{'id': '6a1c7f34-aef7-11eb-b075-1e008a366d48', 'mimeType': 'text/plain', 'text': 'hello world'}
```

```python
d.binary_str()
```

```
b'\n$6a1c7f34-aef7-11eb-b075-1e008a366d48R\ntext/plainj\x0bhello world'
```

### Add Recursion to `Document`

You can add **chunks** (sub-document) and **matches** (neighbour-document) to a `Document` via the following ways:

- Add in constructor:

  ```python
  d = Document(chunks=[Document(), Document()], matches=[Document(), Document()])
  ```

- Add to existing `Document`:

  ```python
  d = Document()
  d.chunks = [Document(), Document()]
  d.matches = [Document(), Document()]
  ```

- Add to existing `doc.chunks` or `doc.matches`:

  ```python
  d = Document()
  d.chunks.append(Document())
  d.matches.append(Document())
  ```

### Visualize `Document`

To better see the Document's recursive structure, you can use `.plot()` function. If you are using JupyterLab/Notebook,
all `Document` objects will be auto-rendered.

<table>
  <tr>
    <td>

```python
import numpy as np
from jina import Document

d0 = Document(id='🐲', embedding=np.array([0, 0]))
d1 = Document(id='🐦', embedding=np.array([1, 0]))
d2 = Document(id='🐢', embedding=np.array([0, 1]))
d3 = Document(id='🐯', embedding=np.array([1, 1]))

d0.chunks.append(d1)
d0.chunks[0].chunks.append(d2)
d0.matches.append(d3)

d0.plot()  # simply `d0` on JupyterLab
```

</td>
<td>
<img src="https://github.com/jina-ai/jina/blob/master/.github/images/four-symbol-docs.svg?raw=true"/>
</td>
</tr>
</table>

## `DocumentArray` API

```python
from jina import Executor, requests, DocumentArray


class MyExec(Executor):

  @requests
  def foo(self, docs: DocumentArray, **kwargs) -> Optional[DocumentArray]:
    ...
```

## Extracting Multiple Attributes

One can extract multiple attributes from a `Document` via


