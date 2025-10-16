#  Aula: Amazon CloudFormation

##  Tema
**Automação de recursos na AWS utilizando CloudFormation**

---

## Anotações da Aula

### Prática Realizada
1. **Acesso à ferramenta:**  
   - Entramos no **AWS Management Console** e acessamos o serviço **CloudFormation**.

2. **Criação de um Stack:**  
   - Selecionamos a opção **“Create Stack”**.  
   - Escolhemos a opção de **utilizar um template existente**.

3. **Upload do Template:**  
   - Fizemos o **upload de um template pré-pronto** para criação de uma instância **EC2**.  
   - A plataforma interpretou o template e solicitou um **nome para o stack**.

4. **Execução e Criação do Recurso:**  
   - Após o deploy do stack, a instância **EC2** foi criada automaticamente.  
   - Subimos o **Apache Server** na instância criada.

5. **Configuração de Segurança (Firewall):**  
   - Criamos uma regra para **liberar apenas a porta 80**, garantindo acesso web e maior segurança.

---

##  Insights e Aprendizados

- O **CloudFormation** **simplifica a criação e o gerenciamento de recursos AWS** através de **templates declarativos (YAML/JSON)**.  
- Esses templates permitem **automatizar a infraestrutura**, reduzindo erros manuais e garantindo consistência.  
- É possível **definir parâmetros e variáveis** para personalizar os recursos criados.  
- **Stacks nomeados** ajudam na **organização e rastreabilidade** dos recursos.  
- A ferramenta segue o conceito de **Infraestrutura como Código (IaC)**, semelhante ao Terraform e a outros orquestradores.  

---

## 🔍 Observação Pessoal
Durante a prática, ficou evidente como o CloudFormation **reduz a complexidade operacional** de criar recursos manualmente.  
A estrutura baseada em templates torna o processo **reutilizável, rápido e padronizado**, ideal para ambientes corporativos e automação em larga escala.
