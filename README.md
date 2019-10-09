# Public Transport SP - Graph Database
Metropolitan Transport Network from São Paulo, Brazil, mapped in a NoSQL graph database.

Read this in other languages: [Português](README.pt-br.md)

You can read a more detailed explanation (in portuguese) in my medium: https://medium.com/@igorrozani/transporte-de-sp-em-um-banco-de-grafos-3ce17d4f1f41

## Summary
* [Inspiration](#inspiration)
* [Database structure](#database-structure)
* [Required softwares](#required-softwares)
* [Examples](#examples)
  * [Create](#create)
  * [Delete](#delete)
  * [Queries](#queries)

## Inspiration

Based in the metropolitan transport network map from São Paulo (image below), i mapped all the stations, terminals, lines and connections utilizing the database Neo4j and the language openCypher.

![Map](img/map.png?raw=true "Map")

## Database structure
It was defined the following database model:


![Database model](img/TransportSP.png?raw=true "Database model")

The nodes are:
- BusTerminal - bus terminals. Example: São Bernardo;
- Company - the company responsable by the transport line. Example: CPTM;
- Line - the transport lines. Example: Yellow line;
- MetroStation - the metro stations. Example: Paulista;
- OrcaShuttleTerminal - Orca Shuttle terminals. Example: Jabaquara;
- PointOfInterest - points of interest from the map. Example: Zoo;
- TouristicTerminal - tourist terminal stations. Example: Paranapiacaba;
- TrainStation - train stations. Example: Morumbi.

And the relationships are:
- Connect - connection between stations and/or terminals;
- Has - relationship between Station/Terminal and the PointOfInterest;
- Integration - integration between different types of stations and terminals;
- Own - relationship between the company and line, to represent that the company owns that line;
- Part_Of - relationship between the station/terminal and line, to represent that the station/terminal is part of that line.

## Required softwares
To run this project, it's necessary to install the Neo4j, [you can download it here](https://neo4j.com/download).

## Examples

### Create

#### Create a node

```
CREATE (:Line {name:'Emerald', number:9})
```

#### Create a relationship

```
MATCH (s1:TrainStation{name:'Poá'}),(s2:TrainStation{name:'Suzano'})
CREATE (s1)-[r:Connect{transport: 'train'}]->(s2)
```

### Delete

#### Delete all database items

```
MATCH (n)-[r]-()
DELETE n,r
```

### Queries

#### All database nodes

```
MATCH (n)
RETURN n
```

#### All stations/terminals from a line

```
MATCH (l:Line)-[:Part_Of]-(s)
RETURN l.name AS Line, collect(s.name) AS Stations
```

#### All stations with elevator

```
MATCH (s {hasElevator:true})
RETURN s
```

#### All station connections 

```
MATCH ({name:'Luz'})-[:Connect]-(s)
Return s
```

#### All node connections of TouristicTerminal label

```
MATCH (s:TouristicTerminal)-[:Connect]-({name:'Luz'})
Return s
```

#### All lines from a company

```
MATCH (c:Company)-[:Own]-(l:Line)
WITH c, l
ORDER BY l.number, l.name
WITH c, collect(CASE WHEN l.number IS NULL THEN l.name ELSE l.number + ' - ' + l.name END) as lines
RETURN c.name, lines
ORDER BY c.name
```

#### All relationship type sorted by alphabetical order

```
MATCH ()-[r]-()
WITH DISTINCT type(r) AS relationships
RETURN DISTINCT relationships
ORDER BY relationships
```

#### All nodes label sorted by alphabetical order

```
MATCH (n)
WITH DISTINCT labels(n) AS labels
UNWIND labels AS label
RETURN DISTINCT label
ORDER BY label
```

#### Quantity stations or terminals by line

```
MATCH (l:Line)-[:Part_Of]-(s)
WITH l, count(s) as qtd
RETURN l.name, qtd
ORDER BY l.name
```

#### Quantity of place with bike parking terminal or bike attaching post

```
MATCH (n)
WHERE n.hasBikeAttachingPost = true OR n.hasBikeParkingTerminal = true
RETURN n
```

#### All nodes that are in more than one line

```
MATCH (s)-[p:Part_Of]-(:Line)
WITH s, count(p) as qtd
WHERE qtd > 1
RETURN s.name, qtd
ORDER BY qtd DESC
```

#### Shortest path between stations, indepent of stations types

```
MATCH x = shortestPath((s1{name:"Grajaú"})-[:Connect*]-(s2{name:"Rio Grande da Serra"}))
RETURN EXTRACT(n IN NODES(x) | n.name) AS Directions
```

#### Shortest path between stations, only using train or metro

```
MATCH 
	(s1{name:"Grajaú"}), 
	(s2{name:"Rio Grande da Serra"}),
	p = shortestPath((s1)-[:Connect*]-(s2))
WHERE ALL (x IN RELATIONSHIPS(p) WHERE x.transport='train' OR x.transport='metro')
RETURN EXTRACT(n IN NODES(p) | n.name) AS Directions
```
