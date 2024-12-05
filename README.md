# praticando com helm
- Para iniciar essa aplicação em especifica com o minikube, eu startei ele com 6G de memória e 4 núcleos para deixar o cluster local mais rápido
- instalar o helm pelo site oficial: **https://helm.sh/pt/docs/intro/install/**
```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
- Helm charts, a melhor maneira de se pensar sobre é que o helm charts seria basicamente um dockerfile, só que dentro do kubernetes
- **https://artifacthub.io/** um reposítorio de pacotes, como se fosse um dockerhub
- Neste caso, utilizei o pacote e o chart do mysql
```bash 
# adicionando o repoistorio do responsável que criou o mysql
# (helm -> é um comando)
helm repo add bitnami https://charts.bitnami.com/bitnami

# instalando o charts o mysql
helm install my-mysql bitnami/mysql --version 12.1.0
```
- Assim que instala qualquer tipo de charts, vai voltar para o terminal um resumo do que foi feito, algo como:
```bash 
-version 12.1.0 # versão do Helm Chart que foi instalada
NAME: my-mysql # nome da instalação do MySQL
LAST DEPLOYED: Wed Dec  4 13:35:46 2024 # data e hora da última instalação ou atualização
NAMESPACE: default # namespace onde a instalação foi realizada
STATUS: deployed # status atual da instalação, indicando que está em execução
REVISION: 1 # número da revisão da instalação, indicando quantas vezes foi atualizada
TEST SUITE: None # indica que não há testes associados a esta instalação
NOTES: # informações adicionais sobre a instalação (nenhuma informação adicional neste caso)
CHART NAME: mysql # nome do Helm Chart que foi instalado
CHART VERSION: 12.1.0 # versão específica do Helm Chart instalado
APP VERSION: 8.4.3 # versão da aplicação MySQL que está sendo instalada
``` 
- Quando instala um charts, neste caso do mysql, ele já traz algumas configurações prontas, como services, nome e senha de usuario
- para descobrir a senha  padrão que o mysql informa, geralmente aparece no terminal. 
```bash
kubectl get secret --namespace default my-mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d

# senha
NrJv5V1ppq

```
- A vantangem de usar um helm, é que já tras tudo pronto, e neste caso específico do mysql que foi instalado com o helm, não precisei criar arquivos de configuração, secrets e nem nada do tipo pois
o próprio helm já faz isso de forma automática.

### Crinado um chart
- Estrutura de diretórios de um Chart
```bash
mychart/
  Chart.yaml          # Contém informações sobre o chart
  values.yaml         # Valores padrão de configuração do chart
  charts/             # Contém charts dependentes
  templates/          # Arquivos de template que definem os recursos Kubernetes
  templates/_helpers.tpl  # Arquivo de ajuda para templates
  .helmignore         # Arquivo para listar arquivos/padrões a serem ignorados
  ``` 
- **Chart.yaml:** Contém metadados sobre o chart, como nome, versão e descrição

- **values.yaml:** Contém valores de configuração padrão para o chart. Esses valores podem ser substituídos durante a instalação ou atualização do chart.

- **charts/:** Contém charts dependentes, se houver. Esses são charts que são instalados junto com o chart principal.

- **templates/:** Contém arquivos de template que são usados para gerar os manifestos Kubernetes. Exemplos de arquivos de template incluem deployment.yaml, service.yaml, etc.

- **templates/_helpers.tpl:** Contém definições de funções de template que podem ser usadas em outros arquivos de template. Essas funções ajudam a manter os templates DRY (Don't Repeat Yourself).

- **.helmignore:** Funciona de forma semelhante a um .gitignore, especificando os arquivos e diretórios que devem ser ignorados ao empacotar o chart.

### Criando um template
- ConfigMap, este template define a estrutura básica de um ConfigMap, incluindo os campos que permanecem constantes em diferentes instâncias
- Basicamente com o helm é possível reutilizar os templates
- Values, os values é possível adicionar qualquer informação sobre a aplicação, que neste caso é o alura foods
```bash 
#Dentro do values.yml

configMap:  # o value que seŕa usado, poderia ser um deployment, secrets, statefulsets...
  name: dados
  data:
    SERVER_HOST: "http://server-0.server:8081/eureka,http://server-1.server:8081/eureka"
    DB_HOST: "mysql"
```

- Para acessar os values que foi criado, acessar a informação sobre o configmap, criar um template, neste cenário é um configmap.. 
```bash 

#Dentro do configmap.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Values.configMap.name }}
data:
  
```
- **{{ .Values.configMap.name }}** essa linha siginfica que o configmap.yml esta acessando o values.yml, depois que acessa o values, o helm substitui a linha **{{ .Values.configMap.name }}** para os valores que estão no values.yml

- Agora, acessando os valores **data:** de forma automática utilizando um loop do tipo **range**
```bash

```
