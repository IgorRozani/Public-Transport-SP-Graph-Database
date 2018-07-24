# Public Transport SP - Graph Database

Transporte Metropolitano de SP mapeado em um banco de NoSQL de grafos.

Você pode ler uma explicação mais detalhada no meu medium: https://medium.com/@igorrozani/transporte-de-sp-em-um-banco-de-grafos-3ce17d4f1f41

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
CREATE (:Line {name:'Emerald', number:9})
```

#### Criar um relacionamento

```
MATCH (s1:TrainStation{name:'Poá'}),(s2:TrainStation{name:'Suzano'})
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

#### Todos os nós que possuem elevador

```
MATCH (s {hasElevator:true})
RETURN s
```

#### Todas as conexões de um nó

```
MATCH ({name:'Luz'})-[:Connect]-(s)
Return s
```

#### Todas as conexões do label TouristicTerminal de um nó

```
MATCH ({name:'Luz'})-[:Connect]-(s:TouristicTerminal)
Return s
```

#### Todas as linhas de uma empresa

```
MATCH (c:Company)-[:Own]-(s)
RETURN c.name, collect(s.name)
```

#### Todos os tipos de conexão ordernado por ordem alfabética

```
MATCH ()-[r]-()
WITH DISTINCT type(r) AS relationships
RETURN DISTINCT relationships
ORDER BY relationships
```

#### Todos os labels dos nós ordernado por ordem alfabética

```
MATCH (n)
WITH DISTINCT labels(n) AS labels
UNWIND labels AS label
RETURN DISTINCT label
ORDER BY label
```

#### Quantidade de estações/terminais das linhas

```
MATCH (l:Line)-[p:Part_Of]-()
WITH l, count(p) as qtd
RETURN l.name, qtd
ORDER BY qtd DESC
```

#### Quantidade de locais que possuem bicicletário ou paraciclos

```
MATCH (s)
WHERE s.hasBikeParkingTerminal OR s.hasBikeAttachingPost
WITH count(s) as qtd
RETURN qtd
```

#### Todos nós que estão em mais de uma linha

```
MATCH (s)-[p:Part_Of]-(:Line)
WITH s, count(p) as qtd
WHERE qtd > 1
RETURN s.name, qtd
ORDER BY qtd DESC
```

#### Caminho com o menor número de estações independente do tipo

```
MATCH x = shortestPath((s1{name:"Grajaú"})-[:Connect*]-(s2{name:"Rio Grande da Serra"}))
RETURN EXTRACT(n IN NODES(x) | n.name) AS Directions
```

#### Menor caminho utilizando apenas trem ou metro

```
MATCH 
	(s1{name:"Grajaú"}), 
    (s2{name:"Rio Grande da Serra"}),
	p = shortestPath((s1)-[:Connect*]-(s2))
WHERE ALL (x IN RELATIONSHIPS(p) WHERE x.transport='train' OR x.transport='metro')
RETURN EXTRACT(n IN NODES(p) | n.name) AS Directions
```
