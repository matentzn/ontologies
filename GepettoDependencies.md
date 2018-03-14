# Gepetto query dependencies
From [XMI](https://github.com/VirtualFlyBrain/VFB2/blob/master/html/conf/vfb.xmi)

## AberOWL

* FBbt_00007239: http://purl.obolibrary.org/obo/[ID]
* http://purl.obolibrary.org/obo/ : obo

| Label XMI | REST | DL Query |
|-------|-------|-------|
| Part of $NAME | type=subeq&amp;query=%3Cobo:BFO_0000050%3E%20some%20%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | `part of` some FBbt_00007239
 | Neurons with some part here | type=subeq&amp;query=%3Cobo:FBbt_00005106%3E%20that%20%3Cobo:RO_0002131%3E%20some%20%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | neuron that overlaps some FBbt_00007239
 | Neurons with synaptic terminals here | type=subeq&amp;query=%3Cobo:FBbt_00005106%3E%20that%20%3Cobo:RO_0002130%3E%20some%20%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | neuron that `has synaptic terminal in` some FBbt_00007239
 | Neurons with presynaptic terminals here | type=subeq&amp;query=%3Cobo:FBbt_00005106%3E%20that%20%3Cobo:RO_0002113%3E%20some%20%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | neuron that `has presynaptic terminal in` some FBbt_00007239
 | Neurons with postsynaptic terminals here | type=subeq&amp;query=%3Cobo:FBbt_00005106%3E%20that%20%3Cobo:RO_0002110%3E%20some%20%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | neuron that `has postsynaptic terminal in` some FBbt_00007239
 | Neuron classes fasciculating here | type=subeq&amp;query=%3Cobo:FBbt_00005106%3E%20that%20%3Cobo:RO_0002101%3E%20some%20%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | neuron that `fasciculates with` FBbt_00007239
 | tracts in  |  type=subeq&amp;query=%3Cobo:FBbt_00005099%3E%20that%20%3Cobo:RO_0002134%3E%20some%20%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | `neuron projection bundle` that innervates some FBbt_00007239 |
 | Subclasses of $NAME | type=subeq&amp;query=%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | FBbt_00007239
 | Images of neurons with some part here (clustered) | type=realize&amp;query=%3Cobo:C888C3DB-AEFA-447F-BD4C-858DFE33DBE7%3E%20some%20(%3Cobo:FBbt_00005106%3E%20that%20%3Cobo:RO_0002131%3E%20some%20%3Cobo:FBbt_00007239%3E)&amp;ontology=VFB | `has exemplar` some (neuron that overlaps some FBbt_00007239)
 | Images of neurons with some part here | type=realize&amp;query=%3Cobo:FBbt_00005106%3E%20that%20%3Cobo:RO_0002131%3E%20some%20%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | neuron that overlaps some FBbt_00007239
 | Transgenes expressed in | | |
 | Lineage clones found in | type=subeq&amp;query=%3Cobo:FBbt_00007683%3E%20that%20%3Cobo:RO_0002131%3E%20some%20%3Cobo:FBbt_00007239%3E&amp;ontology=VFB | `neuroblast lineage clone` that overlaps some FBbt_00007239 |

## Neo4J

## Get and process 6 example images from Neo4j for class list

```
MATCH (n:Class) WHERE n.short_form IN ['FBbt_00007239','FBbt_00007152']
OPTIONAL MATCH (n)<-[:SUBCLASSOF|INSTANCEOF*..]-(i:Individual)<-[:Related { label : 'depicts' } ]-(j:Individual)-[k:in_register_with]->(m:Individual)
OPTIONAL MATCH (n)-[:SUBCLASSOF]->(c:Class)
RETURN n.short_form as class_Id, COLLECT(DISTINCT c.label) as class_Type, COLLECT (DISTINCT { image_name: i.label, image_id: i.short_form, image_thumb: replace(k.iri,'http:','https:') + '/thumbnailT.png', template_id: m.short_form}) AS inds
LIMIT 6
```

```
MATCH (n:VFB:Class) WHERE n.short_form IN ['FBbt_00007239','FBbt_00007152'] RETURN count(n) AS count
```
### Get and process details from Neo4j for list of inds"

```
MATCH (i:Individual) WHERE i.short_form IN ['FBbt_00007239','FBbt_00007152']
OPTIONAL MATCH (i)<-[:Related {label:'depicts'}]-(:Individual)-[k1:in_register_with]->(:Individual)
OPTIONAL MATCH (i:Cluster)<-[:Related {label:'exemplar_of'}]-(e:Individual)
OPTIONAL MATCH (i)-[:INSTANCEOF]->(ic:Class)
OPTIONAL MATCH (e)-[:INSTANCEOF]->(ec:Class) with coalesce('Exemplar: ' + ec.label,ic.label) as ct, i, coalesce(replace(k1.iri,'http:','https:') + '/thumbnailT.png','http://flybrain.mrc-lmb.cam.ac.uk/vfb/fc/clusterv/3/' + e.label + '/snapshot.png') as iri, i.description[0] as cd
RETURN i.short_form as id,
CASE
WHEN not i.synonym is null
THEN i.label+replace(' ('+reduce(a='',n in i.synonym|a+n+', ')+')',', )',')')
ELSE i.label
END as name, cd as def, COLLECT(DISTINCT ct) as type, iri AS file
```

```
MATCH(i:Individual) WHERE i.short_form IN ['FBbt_00007239','FBbt_00007152'] RETURN count(i) as count
```

### Get fellow cluster members"

```
MATCH (n:Neuron { short_form: 'FBbt_00007239' } )-[r1:Related {label:'member_of'}]->(c:Cluster)-[r2:Related {label:'has_member'}]->(i:Neuron)<-[:Related { label : 'depicts' } ]-(j:Individual)-[k:in_register_with]->(m:Individual)
OPTIONAL MATCH (i)-[:INSTANCEOF]->(ec:Class)
RETURN i.short_form as id,
CASE
WHEN not i.synonym is null
THEN i.label+replace(' ('+reduce(a='',n in i.synonym|a+n+', ')+')',', )',')')
ELSE i.label
END as name, i.description[0] as def, COLLECT(DISTINCT ec.label) as type, replace(k.iri,'http:','https:') + '/thumbnailT.png' AS file
```

```
MATCH (n:Neuron { short_form: 'FBbt_00007239' } )-[r1:Related {label:'member_of'}]->(c:Cluster)-[r2:Related {label:'has_member'}]->(i:Neuron) RETURN count(i) as count
```

### All example images for a class

```
MATCH p=(n:Class { short_form: 'FBbt_00007239' } )<-[r:SUBCLASSOF|INSTANCEOF*..]-(i:Individual)<-[:Related { label : 'depicts' } ]-(j:Individual)-[k:in_register_with]->(m:Individual)
WITH i, k ORDER BY length(p) asc
OPTIONAL MATCH (i)-[:INSTANCEOF]->(c:Class)
RETURN distinct i.short_form as id, CASE WHEN not i.synonym is null THEN i.label+replace(' ('+reduce(a='',n in i.synonym|a+n+', ')+')',', )',')') ELSE i.label END as name, i.description[0] as def, COLLECT(DISTINCT c.label) as type, replace(k.iri,'http:','https:') + '/thumbnailT.png' AS file
```

```
MATCH (n:VFB:Class { short_form: 'FBtp0106654' } )<-[r:SUBCLASSOF|INSTANCEOF*..]-(i:Individual) RETURN count(i) as count
```

### Find domains for template

```
MATCH (n:Template {short_form:'FBbt_00007239'})<-[:Related {label:'depicts'}]-(:Template)<-[r:in_register_with]-(dc:Individual)-[:Related {label:'depicts'}]->(di:Individual)-[:INSTANCEOF]->(d:Class)
WHERE has(r.index)
OPTIONAL MATCH (d)-[:SUBCLASSOF]->(pc:Class)
RETURN distinct di.short_form as id, d.label + ' (' + di.label + ')' as name, d.description[0] as def, COLLECT(DISTINCT pc.label) as type, replace(r.iri,'http:','https:') + '/thumbnailT.png' AS file
```

```
MATCH (n:Template {short_form:'FBbt_00007239'})<-[:Related {label:'depicts'}]-(:Template)<-[r:in_register_with]-(dc:Individual)-[:Related {label:'depicts'}]->(di:Individual)-[:INSTANCEOF]->(d:Class)
WHERE has(r.index) RETURN count(di) as count
```

### Get cluster members

```
MATCH (c:Cluster { short_form: 'FBbt_00007239' } )-[r2:Related {label:'has_member'}]->(i:Neuron)<-[:Related { label : 'depicts' } ]-(j:Individual)-[k:in_register_with]->(m:Individual)
OPTIONAL MATCH (i)-[:INSTANCEOF]->(ec:Class)
RETURN i.short_form as id,
CASE
WHEN not i.synonym is null
THEN i.label+replace(' ('+reduce(a='',n in i.synonym|a+n+', ')+')',', )',')')
ELSE i.label END as name, i.description[0] as def, COLLECT(DISTINCT ec.label) as type, replace(k.iri,'http:','https:') + '/thumbnailT.png' AS file
```

```
MATCH (c:Cluster { short_form: 'FBbt_00007239' } )-[r2:Related {label:'has_member'}]->(i:Neuron)
RETURN count(i) as count
```

### Get term info

```
MATCH (n {short_form:'FBbt_00007239'})
OPTIONAL MATCH (n)-[r]-(o)
WITH n, r, o, startnode(r) as s
WITH n, r, o,
CASE when s.iri = n.iri then 'node' else 'to' end as starts
WHERE (NOT type(r) = 'REFERSTO') AND (starts = 'node' OR r.label = 'depicts' OR r.label = 'is a' OR type(r) = 'INSTANCEOF')
OPTIONAL MATCH (n:Class)<-[r:SUBCLASSOF]-(o:Class)<-[fr:INSTANCEOF]-(fo:Individual)<-[:Related {label:'depicts'}]-(:Individual)-[i0:in_register_with]->(:Individual)-[:Related {label:'depicts'}]->(t0:Individual)
OPTIONAL MATCH (o:Individual)<-[:Related {label:'depicts'}]-(:Individual)-[i1:in_register_with]->(:Individual)-[:Related {label:'depicts'}]->(t1:Individual)
OPTIONAL MATCH (o:Individual)-[i2:in_register_with]->(:Individual)-[:Related {label:'depicts'}]->(t2:Individual)
OPTIONAL MATCH (o:Individual)<-[i3:in_register_with]-(:Individual)-[:Related {label:'depicts'}]->(t3:Individual)-[:INSTANCEOF {label:'type'}]->(dc:Class)
WHERE has(i3.index)
WITH  n, r, starts, coalesce(i1,i3,i2,i0) as i, coalesce(t1,t3,t2,t0) as t, dc, o, fr, fo
ORDER BY t.short_form ASC
RETURN 0 as order, labels(n) as labels, n as node, COLLECT({edge:coalesce(fr,r),types:type(coalesce(fr,r)),start:starts,to:coalesce(fo,dc,o),labels:labels(coalesce(fo,o)),tempIm:i,temp:t}) as links
UNION ALL MATCH (pi:Individual:Synaptic_neuropil {short_form:'FBbt_00007239'})-[pr:INSTANCEOF {label:'type'}]->(n:Class)
MATCH (n)-[r]->(o)
WHERE NOT type(r) = 'REFERSTO'
RETURN 1 as order, labels(n) as labels, n as node, COLLECT({edge:r,types:type(r),start:'node',to:o,labels:labels(o),tempIm:null,temp:null}) as links
ORDER BY order ASC
UNION ALL MATCH (pi:Individual:Cluster {short_form:'FBbt_00007239'})-[pr:Related {label:'has_exemplar'}]->(n:Individual)
MATCH (n)-[r]->(o) WHERE NOT type(r) = 'REFERSTO'
RETURN 1 as order, labels(n) as labels, n as node, COLLECT({edge:r,types:type(r),start:'node',to:o,labels:labels(o),tempIm:null,temp:null}) as links
ORDER BY order ASC
```

```
MATCH (n { short_form: 'FBbt_00007239' } ) RETURN count(n) as count
```
