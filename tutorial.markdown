---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: "SFES Doc | Tutorial"
permalink: /tutorial
---

##### Navegação

- [Voltar ao início](./)
- [Limitações](./limitacoes)

---

## Passo a passo

##### Vamos simular um projeto para exemplo de integração:
1. Crie uma Collection no Postman com todos os endpoints passados pelo cliente;
2. Teste todos os endpoints e retornos no Postman;
3. Salve os tipos de resposta existentes (200, 201, 404, 500, etc.) para cada endpoint a ser utilizado. Isso fará com que a definição do [OpenAPI](https://help.salesforce.com/s/articleView?id=sf.external_services_examples_openapi_3_0.htm&type=5) crie corretamente o corpo de resposta que esperamos para tratarmos os dados;
4. Com o Postman testado, identifique e separe a quantidade de domínios existentes nas integrações - às vezes o domínio/DNS de autenticação é diferente dos endpoints de consumo, exemplo: https://google.com, https://salesforce.com, https://login.salesforce.com.
5. !!!***Para cada***!!! domínio iremos criar uma Credencial Nomeada (NamedCredential) que naturalmente necessita de uma Credenciais externas (ExternalCredential) - configuração que é utilizada para inserir dados sensíveis de autenticação como tokens e senhas:
	1. Para criar, no Salesforce acesse Configuração -> Credenciais Nomeadas (Named Credentials) -> Credencial externa (External Credentials) -> Novo;
		1. Rótulo, recomendo `<nome_domínio>_EC`;
		2. Protocolo de autenticação, Personalizado;
		3. Na interna da Credencial externa, clique em Criar na seção Entidades de segurança (Principals, em inglês);
			1. Nome do parâmetro, pode ser "*Auth*";
				1. Ponto de atenção: caso os endpoints a serem utilizados exclusivamente por esse domínio que está atualmente configurando exija cabeçalhos (Headers na requisição), insira seus valores sensíveis clicando em Adicionar em Parâmetros de autenticação e então preencha com o nome que desejar pois é esse nome que inseriremos **depois** no Header;
			2. Número sequencial mantenha 1;
			3. Tipo de identidade, mantenha *Principal nomeado*;
			4. Salvar;
	2. Agora, contendo o registro da *Entidade de segurança (Principals)*, note duas coisas ao clicar em editar:
		1. Os valores dos *Parâmetros de autenticação* estão vazios. **Por segurança o Salesforce oculta esses valores e é importante que eles estejam salvos em outro lugar para consulta, e não dentro do Salesforce**;
		2. A frase *"Acesso da entidade de segurança. Você não tem nada associado a essa entidade de segurança."* aparece. Isso significa que nenhum Conjunto de permissão (PermissionSet) foi vinculado ao Principals. Vamos criar um:
			1. Configurações -> Conjunto de permissões (Permission Sets) -> Novo;
			2. Label, recomendo `<nome_credencial_externa>_PS`;
			3. Licença, geralmente "Salesforce" ou "Salesforce Integration" (caso use);
			4. Salvar;
			5. Ainda na interna do Conjunto de permissões, na seção Aplicativos procure por "Acesso da entidade de segurança da credencial externa" (External Credential Principal Access);
			6. Clique em Editar, selecione sua Credencial Externa que criou e Salve;
			7. Ainda, acima clique em Gerenciar atribuições e adicione o usuário ao qual irá executar as requisições;
			8. Vamos voltar a tela da Credencial Externa;
		3. Agora ao editar a Entidade de segurança deve aparecer nosso Conjunto de Permissões atribuído;
		4. (Situacional): Repita o passo 5.1 em diante para cada Domínio que existir na Collection do Postman. Se achar correto, pode utilizar o mesmo Conjunto de permissão para a nova Credencial Externa que for criar. (Por que criar outra? Pois é **possível** que nesse novo domínio hajam outros parâmetros de autenticação sensíveis. Caso não haja, utilize a mesma Credencial Externa na nova Credencial Nomeada se desejar);
6. Agora podemos criar a nossa Credencial Nomeada. Configurações -> Credenciais Nomeadas -> Novo;
	1. Insira a URL apenas do domínio sem `/` no fim;
	2. Selecione a Credencial externa que criou para esta Credencial Nomeada;
	3. (Situacional) Apenas seleciona "Permitir fórmulas no cabeçalho HTTP" se você for montar Headers de AuthBasic (aqueles que convertemos usuário e senha num base64, pois poderemos utilizar um comando para pegar os *Parâmetros de autenticação* e gerarmos ali mesmo o base64);
	4. Deixe o restante padrão e clique em Salvar;
7. Agora chegou a hora de criarmos o ExternalService (Serviço externo);
	1. Configuração -> Serviços externos (External services) -> Adicionar um serviço externo -> Da especificação da API;
		1. Nome do serviço, recomendo `<nome_dominio>_ES`;
		2. Esquema de serviço: JSON completo;
		3. Selecionar uma credencial nomeada: a do domínio que queremos configurar;
		4. Como conseguir o JSON:
			1. No Postman, separe os domínios em Collections diferentes para cada Credencial Nomeada que criou;
			2. (Situacional) Nas respostas dos endpoints que você salvou, verifique se a resposta não está muito grande, pois não há necessidade de enviar a resposta inteira. Ela serve apenas como exemplo de resposta para poder ser acessada na resposta da Action do Fluxo. Basicamente essas respostas de requisição do Postman vão servir como as Classes de ResponseTemplate que geralmente criamos para integrar;
			3. Agora na Collection que estamos configurando: botão direito do mouse -> Export -> Export;
			4. Copie o JSON exportado pelo Postman e utilize o seguinte site para convertê-lo para o formato [[OpenAPI]]: [https://kevinswiber.github.io/postman2openapi/](https://kevinswiber.github.io/postman2openapi/);
			5. Copie o formato [[OpenAPI]] e cole no [[Swagger]]: [https://editor-next.swagger.io/](https://editor-next.swagger.io/);
			6. Verifique se há erros e se a URL do servidor está correta como a do Postman (pode ser que URLs com Portas não sejam copiadas);
			7. Clique em File -> Convert and Save as JSON;
			8. Utilize esse JSON;
		5. Se ocorrer algum erro ao colar o JSON e fizer algum ajuste, clique em Validar;
		6. Avance, verifique e selecione os endpoints que irão ser criados a partir da sua definição OpenAPI;
	2. Pronto! Agora poderemos utilizá-los finalmente em todos os tipos de Fluxos e até mesmos classes Apex utilizando o Namespace, exemplo: `new ExternalService.BrwSapES();`
8. Vamos utilizar num Fluxo:
	1. Coloque um elemento Action na tela;
		1. Se quiser visualizar todos os Endpoints disponíveis por ExternalServices, clique na esquerda em: Categoria -> Tipo -> External Service. E então ao clicar no campo de busca aparecerão todos os endpoints cadastrados na [[Organizações (orgs)|org]];
		2. Após inserir, note que aparecerão os valores a serem preenchidos para executar a requisição, como os parâmetros de URL, de Header e/ou de body, caso tenham sido configurados;
		3. Para inserirmos dados no body precisamos utilizar uma variável que o próprio ExternalService criou ao criarmos sua definição:
			1. Crie um novo recurso -> Variável -> Tipo de dado -> **Apex-Defined** -> pesquise pela classe que representa exatamente o body de sua requisição, ex: `ExternalService__BrwSapES_login_IN_body`;
	2. Agora é possível utilizar os dados de resposta das requisições das Actions a vontade, tanto em Flow como em classes Apex já com a configuração e autenticação previamente definida.

> Ref: Duvido achar documentação melhor.