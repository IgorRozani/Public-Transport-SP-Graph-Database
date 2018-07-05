# Public Transport SP - Graph Database

Transporte Metropolitano de SP mapeado em um banco de NoSQL de grafos.

## Sumário

* [Inspiração](#inspiração)
* [Estrutura do banco](#estrutura-do-banco)
* [Softwares necessários](#softwares-necessários)
* [Exemplos](#exemplos)
  * [Criação](#criação)
  * [Exclusão](#exclusão)
  * [Consultas](#consultas)

## Inspiração

Baseado no mapa da rede de transporte metropolitano da cidade de São Paulo (imagem abaixo), mapeei todas as estações, terminais, linhas e conexões utilizando em um banco NoSQL de grafos, utilizando o banco Neo4j e a linguagem openCypher.

![Map](img/map.png?raw=true "Map")

## Estrutura do banco

Baseado na imagem anterior, foi definida a seguinte modelagem do banco:

![Database model](img/TransportSP.png?raw=true "Database model")

Sendo composto dos seguintes labels de nós (nodes), cada um representando um ponto do mapa:
- BusTerminal - terminais de ônibus. Por Exemplo: São Bernardo e Jabaquara;
- Company - empresas responsáveis pelas linhas de transporte de SP. Exemplo: CPTM;
- Line - linhas de transporte. Exemplo: Linha amarela;
- MetroStation - estações de metro. Exemplo: Estação Paulista;
- OrcaShuttleTerminal - terminais da Ponte Orca. Exemplo: Estação Jabaquara;
- PointOfInterest - Pontos de interesse. Exemplo: Rodoviária e Zoológico;
- TouristicTerminal - estações turística. Exempo: Estação Paranapiacaba;
- TrainStation - estações de trem. Exemplo: Morumbi.

E dos seguintes relacionamentos:
- Connect - conexão entre estações e/ou terminais;
- Has - localização de um ponto de interesse;
- Integration - integração entre diferentes tipos de estações e/ou terminais;
- Own - posse de um linha por uma empresa;
- Part_Of - a participação de um estação e/ou terminal em uma linha.

## Softwares necessários

Para rodar o projeto, é necessário instalar o Neo4j, você pode baixar [clicando aqui](https://neo4j.com/download/?ref=hro).

## Exemplos

### Criação

#### Criar um nó

```
CREATE (:Line {name: 'Esmeralda', company: 'CPTM', number: 9})
```

#### Criar um relacionamento

```
MATCH (s1:Station),(s2:Station)
WHERE s1.name = 'Poá' AND s2.name = 'Suzano'
CREATE (s1)-[r:Connect{transport: 'train'}]->(s2)
```

### Exclusão

#### Excluir todos os itens do banco

```
MATCH (n)-[r]-()
DELETE n,r
```

### Consultas

#### Visualizar todos nós do sistema

```
MATCH (n)
RETURN n
```

#### Todas as estações de uma linha

```
MATCH (:Line{name:'Jade'})-[:Has]-(s:Station)
RETURN s
```

#### Todas as estações que possuem terminal rodoviário

```
MATCH (s:Station {hasRoadTerminal:true})
RETURN s
```

#### Todas as conexões de uma estação

```
MATCH (s:Station)-[c:Connect]-(s2:Station)
WHERE s.name='Paulista - Consolação'
Return s2
```

#### Todas as linhas de uma empresa

```
MATCH (c:Company)-[:Own]-(s)
RETURN c.name, collect(s.name)
```

#### Todos os tipos de conexão

```
MATCH ()-[r]-()
WITH DISTINCT type(r) AS relationships
RETURN DISTINCT relationships
ORDER BY relationships
```

#### Todos os labels dos nós

```
MATCH (n)
WITH DISTINCT labels(n) AS labels
UNWIND labels AS label
RETURN DISTINCT label
ORDER BY label
```

#### Quantidade de estações das linhas

```
MATCH (l:Line)-[h:Has]-()
WITH l, count(h) as qtd
RETURN l.number, l.name, qtd
ORDER BY l.number
```

#### Quantidade de estações que possuem bicicletário ou paraciclos

```
MATCH (s:Station)
WHERE s.hasBikeParkingTerminal OR s.hasBikeAttachingPost
WITH count(s) as qtd
RETURN qtd
```

#### Todas as estações que estão em mais de uma linha

```
MATCH (s:Station)-[h:Has]-(:Line)
WITH s, count(h) as qtd
WHERE qtd > 1
RETURN s.name, qtd
ORDER BY qtd DESC
```

#### Caminho com o menor número de estações

```
MATCH x = shortestPath((s1:Station)-[:Connect*]-(s2:Station))
WHERE s1.name="Grajaú" AND s2.name="Rio Grande da Serra"
RETURN EXTRACT(n IN NODES(x) | n.name) AS Directions
```

#### Menor caminho utilizando apenas o trem

```
MATCH (s1:Station {name:'Pinheiros'}),
	    (s2:Station {name:'Corinthians - Itaquera'}),
      p = shortestPath((s1)-[:Connect*]-(s2))
WHERE ALL (x IN RELATIONSHIPS(p) WHERE x.transport='train')
RETURN p
```
