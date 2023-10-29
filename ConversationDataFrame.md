:dart: **Objetivo:** Você possui uma tabela com as conversas entre Empresas e Usuários e  precisa transformar essa tabela em uma tabela com as datas das conversas, as empresas, os clientes e todas as conversas agrupadas em apenas uma coluna de maneira ordenada. Exemplo da tabela:
:dart: **Goal:** You have a table with conversations between Companies and Clients, you need to transform this table into a table with the dates of conversations, companies, customers and all conversations grouped in just one column in an orderly manner. Table example:

![Table](https://raw.githubusercontent.com/Annallisboa/SQL-Analysis/main/TabeladeConversas.png)

**- 1º Passo/ 1º Step:**
- Unir as queries para termos em apenas uma coluna as Empresas e os Clientes e também uma coluna com Quem Enviou as Mensagens:
- Merge the queries so that we have Companies and Clients in just one column and also a column with Who Sent the Messages:

```SQL
SELECT
	DATE,
	[FROM] AS Company,
	[TO] AS Client,
	CONVERSATION,
	CASE 
	WHEN LEFT([FROM],7) = 'Company' THEN 'Company'
	ELSE 'Client' end as 'WhoSent'
FROM [dbo].[data_conversation]
WHERE LEFT([FROM],7) = 'Company'
	UNION
SELECT
	DATE,
	[TO] AS Company,
	[FROM] AS Client,
	CONVERSATION,
	CASE 
	WHEN LEFT([FROM],7) = 'Company' THEN 'Company'
	ELSE 'Client' end as 'WhoSent'
FROM [dbo].[data_conversation]
WHERE LEFT([TO],7) = 'Company'
```

**2º Passo / 2º Step:**
- Depois disso, criei uma CTE com as tabelas unidas e utlizei as funções de inteligencia de tempo row_number() e LEAD para order as conversas:
- After that, I created a CTE with the joined tables and used the row_number() and Lead() time intelligence functions to order the conversations:
  
```SQL
WITH CONVERSATIONS AS (SELECT
	DATE,
	[FROM] AS Company,
	[TO] AS Client,
	CONVERSATION,
	CASE 
	WHEN LEFT([FROM],7) = 'Company' THEN 'Company'
	ELSE 'Client' end as 'WhoSent'
FROM [dbo].[data_conversation]
WHERE LEFT([FROM],7) = 'Company'
	UNION
SELECT
	DATE,
	[TO] AS Company,
	[FROM] AS Client,
	CONVERSATION,
	CASE 
	WHEN LEFT([FROM],7) = 'Company' THEN 'Company'
	ELSE 'Client' end as 'WhoSent'
FROM [dbo].[data_conversation]
WHERE LEFT([TO],7) = 'Company'
)
SELECT
	date,
	 Company, 
	 Client,
	 WhoSent,
     Conversation,
	 ROW_NUMBER() OVER (PARTITION BY Company, Client ORDER BY DATE) AS LineOrder,
     LEAD(WhoSent,1) OVER (PARTITION BY Company, Client ORDER BY DATE) AS WhoSentNextMessage
FROM CONVERSATIONS
```
O resultado deve ser:
The result must be: 

![Table](https://raw.githubusercontent.com/Annallisboa/SQL-Analysis/main/ConversationOrdered.png)

**Último Passo /Final Step** 
- A query final, nesse caso, precisa ser a data (apenas com dia, mes e ano), Company, Client e agregação de toda a conversa ordernada em apenas 1 linha da tabela (foi criada uma view da tabela anterior chamada CONVERSATIONS): 
- The final query, in this case, needs to be the date (with only day, month and year), Company, Client and aggregation of the entire conversation ordered in just 1 row of the table (a view of the previous table called CONVERSATIONS was created):
 
```SQL 
SELECT
	FORMAT(date,'dd/MM/yyyy', 'en-US') AS Date,
	Company,
	Client,
	string_agg(concat('-', WhoSent, ': ', Conversation), ' ') as All_Conversation
FROM CONVERSATIONS
 group by FORMAT(date,'dd/MM/yyyy', 'en-US'),Company, Client
```

![Table](https://raw.githubusercontent.com/Annallisboa/SQL-Analysis/main/Final%20table.png)
