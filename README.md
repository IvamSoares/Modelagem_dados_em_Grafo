Modelagem de Dados em Grafos de um Serviço de Streaming

📌 Contexto do Projeto
Este projeto nasceu do desafio de mapear conexões complexas entre usuários e a indústria do entretenimento.
Em vez de usar tabelas estáticas, utilizei Bancos de Dados em Grafo (Neo4j) para criar uma rede onde filmes, séries, atores e diretores se conectam de forma dinâmica, 
permitindo recomendações que vão além do "quem viu isso, também viu aquilo".

🛠️ Por que Grafos (Neo4j)?
A escolha por grafos foi motivada pela agilidade nas conexões. Em um banco relacional, descobrir se um usuário gosta de um ator que frequentemente trabalha com um diretor específico exigiria múltiplos JOINs,
o que prejudica a performance. No Neo4j, isso é apenas um salto entre nós.

Desafios e Aprendizados
Durante a modelagem, acabei cometendo alguns erros durante o processo, mas nada que comprometesse o desenvolvimento e entrega do proejto.
No início, tentei declarar ratings usando a mesma variável w no mesmo script. O Neo4j me barrou (Variable already defined). 
Solucionei utilizando o comando WITH para passar o contexto do usuário e renomeando variáveis de relacionamento (r1, r2) para isolar cada operação.

Integridade de Dados: O uso do CREATE gerava duplicatas toda vez que o script rodava. Implementei o MERGE, garantindo que o banco fosse atualizado em vez de clonado.

Conexões Órfãs: Percebi que as séries estavam sem se relacionar com os diretores. 
Resolução: Corrigi o modelo para garantir que tanto filmes quanto séries estivessem sob a mesma hierarquia de Directed e In_genre, permitindo recomendações cruzadas.

📐 Modelo do Grafo
Abaixo, a representação visual da nossa arquitetura de dados:

Labels (Nós): User, Movie, Series, Actor, Director, Genre.

Relacionamentos (Arestas):

WATCHED (propriedade: rating)

ACTED_IN

DIRECTED

IN_GENRE

💾 Scripts de Carga
Abaixo, uma amostra do script Cypher utilizado. Uso do MERGE e SET para garantir que o ano do filme e a nota do usuário sejam atualizados sem criar nós extras.

// Cadastro de Diretor e Obra
MERGE (d:Director {name: 'Christopher Nolan'})
MERGE (m:Movie {title: 'Inception'}) 
SET m.year = 2010
MERGE (d)-[:DIRECTED]->(m)

// Cadastro do Rating
MATCH (u:User {name: 'Eduardo'}), (m:Movie {title: 'Inception'})
MERGE (u)-[r:WATCHED]->(m)
ON MATCH SET r.rating = 5
ON CREATE SET r.rating = 5

🚀 Queries de Negócio (Insights)
O valor real do grafo está em responder perguntas complexas de mercado:

1. Recomendação por "Assinatura do Diretor"
"Sugira filmes para o Eduardo baseados nos diretores das séries que ele deu nota 5."

MATCH (u:User {name: 'Eduardo'})-[r:WATCHED]->(s:Series)<-[:DIRECTED]-(d:Director)
MATCH (d)-[:DIRECTED]->(m:Movie)
WHERE r.rating = 5 AND NOT (u)-[:WATCHED]->(m)
RETURN m.title AS Sugestao, d.name AS Diretor

2. Descoberta de Elenco (Network)
"Quais atores participaram tanto de filmes quanto de séries no nosso catálogo?"

MATCH (a:Actor)-[:ACTED_IN]->(m:Movie), (a)-[:ACTED_IN]->(s:Series)
RETURN a.name, m.title, s.title

📈 Conclusão e Evidências
O modelo provou ser eficiente para Cross-Selling (recomendar séries para quem consome filmes) e para identificar nichos de interesse baseados na equipe técnica, não apenas no título da obra.



🌐 Acesso ao Banco de Dados (Live Demo)

O projeto está hospedado no Neo4j AuraDB. Para visualizar o grafo em tempo real:

- **URI:** neo4j+s://73aa32cb.databases.neo4j.io
As credências de acesso foram inseridas no escopo da entrega do projeto.
