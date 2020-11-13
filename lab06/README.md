## Tarefa de Cypher e o FDA Adverse Event Reporting System (FAERS)

## Exercício 1
Escreva uma sentença em Cypher que crie o medicamento de nome `Metamizole`, código no DrugBank `DB04817`.

### Resolução
~~~cypher
CREATE (:Drug {drugbank: "DB04817", name:"Metamizole"})
~~~


## Exercício 2
Considerando que a `Dipyrone` e `Metamizole` são o mesmo medicamento com nomes diferentes, crie uma aresta com o rótulo `:SameAs` que ligue os dois.

### Resolução
~~~cypher
MATCH (d:Drug {name:"Dipyrone"})
MATCH (p:Drug {name:"Metamizole"})
CREATE (d)-[S:SameAs]->(p)
~~~


## Exercício 3
Use o `DELETE` para excluir o relacionamento que você criou (apenas ele).

### Resolução
~~~cypher
MATCH ()-[S]->()
DELETE S
~~~

## Exercício 4
Faça a projeção em relação a Patologia, ou seja, conecte patologias que são tratadas pela mesma droga.

### Resolução
~~~cypher
MATCH (p1:Pathology)<-[a]-(d:Drug)-[b]->(p2:Pathology)
WHERE a.weight > 20 AND b.weight > 20
MERGE (p1)<-[r:Relates]->(p2)
ON CREATE SET r.weight=1
ON MATCH SET r.weight=r.weight+1
~~~

## Exercício 5
Construa um grafo ligando os medicamentos aos efeitos colaterais (com pesos associados) a partir dos registros das pessoas, ou seja, se uma pessoa usa um medicamento e ela teve um efeito colateral, o medicamento deve ser ligado ao efeito colateral.

### Resolução
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MATCH (d:Drug {code: line.codedrug})
MERGE (p:Person {idperson: line.idperson})
MERGE (p)-[u:Usa]->(d)
ON CREATE SET u.weight=1
ON MATCH SET u.weight=u.weight+1

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS line
MATCH (pa:Pathology {code: line.codePathology})
MERGE (p:Person {id: line.idPerson})
MERGE (p)-[e:Efeito]->(pa)
ON CREATE SET e.weight=1
ON MATCH SET e.weight=e.weight+1

MATCH (d:Drug)-[a]->(p:Person)<-[b]-(pa:Pathology)
WHERE a.weight > 20 AND b.weight > 20
MERGE (d)-[g:Gera]->(pa)
ON CREATE SET g.weight=1
ON MATCH SET g.weight=g.weight+1
~~~

## Exercício 6
Que tipo de análise interessante pode ser feita com esse grafo?
Proponha um tipo de análise e escreva uma sentença em Cypher que realize a análise.

### Resolução
Proposição: Qual efeito colateral para cada droga aconteceu mais de 50 vezes
~~~cypher
MATCH (d)-[g:Gera]->(pa)
WHERE g.weight > 50
RETURN d,pa
~~~

