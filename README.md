# Public Transport SP Graph Database

Baseado no mapa da rede de transporte metropolitano da cidade de São Paulo (imagem abaixo), mapiei todas as estações, linhas e conexões utilizando o banco NoSQL de grafos, utilizando o banco Neo4j.

![Map](img/map.png?raw=true "Map")

## Linhas mapeadas
- [x] Linha 1 - Azul
- [x] Linha 2 - Verde
- [x] Linha 3 - Vermelha
- [x] Linha 4 - Amarela
- [x] Linha 5 - Lilás
- [x] Linha 7 - Rubi
- [x] Linha 8 - Diamante
- [x] Linha 9 - Esmeralda
- [x] Linha 10 - Turquesa
- [x] Linha 11 - Coral
- [x] Linha 12 - Safira
- [x] Linha 13 - Jade
- [x] Linha 15 - Prata

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

#### Visualizar todas estações, linhas e relacionamentos

```
MATCH (n)
RETURN n
```

#### Todas as estações de uma linha

```
MATCH (:Line{name:'Jade'})-[:Has]-(s:Station)
RETURN s
```

#### Todas as conexões de uma estação

```
MATCH (s:Station)-[c:Connect]-(s2:Station)
WHERE s.name='Paulista - Consolação'
Return s2
```

#### Quantidade de estações das linhas

```
MATCH (l:Line)-[h:Has]-()
WITH l, count(h) as qtd
RETURN l.number, l.name, qtd
ORDER BY l.number
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
