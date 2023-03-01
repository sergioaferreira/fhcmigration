
# Migração do Alfresco PT da FHC para a versão 7.2.x

## Abstract

Define-se aqui o plano de migração do Alfresco 5 da FHC para uma versão 7.
Ocorre após terem sido efectuados testes de migração para um ambiente,
denominado *Train*, a funcionar na AWS.

Pretende-se agora fazer a passagem a produção do Alfresco PT (existe
um para cada país, mas os restantes estão fora de âmbito), mas também
a migração do ambiente de QA.

Ao contrário do ambiente *Train*, os dois ambientes a migrar existem na
ClaraNet e não na AWS.

## Dificuldades encontradas na migração de Train

Durante o processo de migração ocorreram as seguintes dificuldades:
* Perda de conectividade à base de dados MariaDB, quando num servidor
diferente do que hospeda os contentores docker. resolvido ao colocar a
base de dados no Host Linux que contem o *docker*.
* Má configuração dos volumes dos contentores, nomeadamente os do SOLR.
Resolvido adicionando os volumes necessários.
* Não configuração dos modelos no Alfresco Share. Resolvido configurando
devidamente o share-config-custom.xml

## Migração do Ambiente QA

??? Nota @Rodrigo: é necessário o caminho de evolução de versões que fizeste
em *Train*

??? @Rodrigo: Vou criar um projecto no github que pertilho para colocar toda a
ficheirada (zips, yamls, dockerfiles e scripts)


Tarefas a executar:
* Criar uma pasta organizada com todas as versões intermédias
(zips e docker-compose.yml+dockerfiles e eventuais scripts)
* Alfresco na nova máquina
  * Criar nova VM Linux, devidamente actualizada e instalar o *Docker*. A cargo da FHC. Estimativa: 1 dia
  * Copiar /opt/alfresco??? para a nova VM (com diferenças de QA e PROD).
  * Clonar base de dados de QA para uma nova instância de MariaDB.
  * Mudar localização da BD (db.url) em alfresco-global.properties para o clone.
  * Alfresco start & Simple Test
  * Estimativa das tarefas anteriores: 2h
* Caminho de migração de versões sem containers. Estimativa: 3h :
  * 5.2.? -> 5.2.? : Instalar o zip da nova versão, actualizando ficheiros de configuração
que existam no classpath (provavelmente em tomcat/shared/classes) e fazer as verificações
(arrancar sem excepções, inserções e pesquisas a funcionar)
  * ?.?.? -> ?.?.? :
* Verificar o docker-compose.yml Estimativa: 3h:
  * Todo o conteúdo do Alfresco (BD,Conteúdo e indíces deve existir em volumes).
  * A cópia do modelo deve fazer parte do dockerfile do repo
  * A configuração do share-config-custom.xml deve ser efectuada no dockerfile do repo
* Migração de versões com containers (Estimativa: 5h):
  * 5.2.? -> 6.0.? : alf_data e indices têm de estar no volume certo dos containers.
*docker compose up*, ver logs sem excepções e testar funcionamento básico.
  * 6.0.? -> 6.1.? :
  * 6.1.? -> 6.2.? :
  * 6.2.? -> 7.0.? :
  * 7.0.? -> 7.1.? :
  * 7.1.? -> 7.2.? :
* Instalação do patch que permite os TMDQ(s). Estimativa: 30min
* Migrar de SOLR4 para SOLR6.x. Estimativa: ???
* Reindexar. Estimativa: 1h
* Verificação Final. Estimativa: 1d

**Nota muito importante: Não saltar versões !!!**

## Migração de Produção

 ??? Dúvida @Rodrigo: Os dockerfiles da pasta do compose neste momento já reflectem a instalação do modelo e dos share-config-custom.xml???

Tarefas a executar:
* Alfresco na nova máquina:
  * Criar nova VM Linux, devidamente actualizada e instalar o *Docker*. A cargo da FHC.
  * Copiar /opt/alfresco??? para a nova VM (com diferenças de QA e PROD). Estimativa: 2min
  * Clonar base de dados de *PROD* para uma nova instância de MariaDB. Estimativa: 30min
IMPORTANTE: fazê-lo com um mysqldump, parando o alfresco para garantir a
coerência
  * Fazer rsync dos conteúdos para nova NAS/volume: Estimativa: ???
  * Mudar localização da BD (db.url) em alfresco-global.properties para o clone. Estimativa: 15min
  * Alfresco start & Test: Estimativa: 2h
* Caminho de migração de versões sem containers. Estimativa: 2h :
  * 5.2.? -> 5.2.? : Instalar o war da nova versão e fazer as verificações
* Verificar o docker-compose.yml.  :
  * Todo o conteúdo do Alfresco (BD,Conteúdo e indíces deve existir em volumes).
  * A cópia do modelo deve fazer parte do dockerfile do repo
  * A configuração do share-config-custom.xml deve ser efectuada no dockerfile do repo
* Migração de versões com containers. Estimativa : 8h :
  * 5.2.? -> 6.0.? :
  * 6.0.? -> 6.1.? :
  * 6.1.? -> 6.2.? :
  * 6.2.? -> 7.0.? :
  * 7.0.? -> 7.1.? :
  * 7.1.? -> 7.2.? :
* Instalação do patch que permite os TMDQ(s). Estimativa: 5h
* Arrancar com container com SOLR6.x (no compose). Estimativa: 2h
* Reindexar: Estimativa: ??? @Rodrigo. Quanto tempo demorou a indexar no Train/AWS ???
* Verificação Final preliminar: Estimativa: 2 dias
* Migração final (inserir documentos criados entre primeira migração e a janela de paragem final):
  * Parar Alfresco 5.2 em PROD
  * rsync de documentos -> É necessário decidir se fazemos ou não sincronização do alf_data
se decidirmos não fazer, era interessante não deixar lixo dos testes. ??? Talvez se possa limpar por ano/mes/dia dentro do alf_data???
  * Clonar base de dados: Estimativa: 30min
  * Executar o Alfresco (os conteúdos inseridos entre a primeira migração e o inicio
da janela de paragem serão agora indexados): Estimativa: 3h
  * Verificação final + Pesquisas a documentos inseridos entre primeira e última cópia da BD - Estimativa: 2h

### Volumes

??? Listar aqui os volumes que vão existir em produção ???

### Containers

??? Listar aqui os containers que vão existir e uma breve explicação da sua função ???

## Verificações e Testes

Sempre que se migrar entre versões de Alfresco, devem ser efectuadas as seguintes
verificações:
* O Alfresco arranca
* Não existem novas excepções no log
* O Alfresco share funciona
* As propriedades especificas da FHC são visíveis no UI do share
* Consegue-se inserir documentos a partir do share, adicionar o aspecto da FHC
e preencher as propriedades ficando o documento disponível ao pesquisar por propriedade
(via node browser)

Na verificação final devem ser efectuadas as seguintes validações adicionais:
* Acesso via CMIS workbench : A partir do CMIS workbench deve ser possível
efectuar query(s) de documentos, preferencialmente documentos com metadados
da FHC
* Acesso via FHNet (aplicação da FHC):
  * Ver lista de documentos de prestador
  * Ver conteúdo de documento do prestador
  * Inserir documento
  * Validar que o documento aparece na lista num prazo razoável (menos de 30s)
  * Emitir facturas (que parte os documentos de um pdf e os insere no alfresco)
* Inserção por lote (Papiro)
  * Inserir lote pequeno e validar que documentos aparecem associados ao prestador
  * Inserir lote grande e verificar se há quebra do serviço (lentidão ou paragem) após inserção
* Testes de carga:
  * Executar inserção massiva com o programa de JMeter, validando que são inseridos
associados a um único prestador
* Connectividade à BD
  * Deixar o Alfresco tranquilo durante 15 horas e verificar que após esse tempo continua
a ter conectividade ao MariaDB remoto

## Referências

* [Upgrading from Alfresco 5.2 to 7.0](https://www.youtube.com/watch?v=kHwq_f9PzYU)
* [Transactional Metdata query](https://docs.alfresco.com/search-services/latest/config/transactional/#configuring-an-optional-patch-for-upgrade)
* [Migrate SOLR4 to SOLR6](https://docs.alfresco.com/search-services/1.3/upgrade/migrate/)