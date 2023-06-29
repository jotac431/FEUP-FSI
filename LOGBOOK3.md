# Trabalho realizado na Semana #3

## Identificação

- Na versão 2019.3.1023, a Progress Telerik UI para ASP.NET AJAX contém uma .NET deserialization vulnerability na função RadAsyncUpload.
- Torna-se possível explorá-la quando se obtém as chaves de encriptação através de outras vulnerabilidades.
- A sua exploração pode resultar em execução de código remota.
- Pode ser prevenida através de uma das definições da aplicação mas, apenas a partir de 2020.1.114, passou a ser uma definição default. 

## Catalogação

- A 9/12/2019, na secção Support do site da Telerik, foi lançada uma página que reportava e indicava maneiras de prevenir este exploit.
- Na página da Telerik sobre o lançamento da versão 2020.1.114, na secção AsyncUpload, é possível ler sobre a definição que foi alterada para default para prevenir a vulnerabilidade.
- Na NDV, o exploit foi publicado a 11/12/2019(Base Score:9.8) sendo que, após análise, foi modificado a 20/10/2020, acrescentando informações relativas à resolução do problema.
- https://twitter.com/mwulftange inicialmente descobriu essa vulnerabilidade. https://github.com/bao7uo escreveu toda a lógica para quebrar a criptografia RadAsyncUpload, que habilitou a manipulação do objeto de configuração de upload de arquivo em rauPostData e subsequentemente explorando a desserialização insegura desse objeto. https://github.com/lesnuages ​​escreveu a primeira iteração do Sliver stager payload.

## Exploit

- As instruções de como reproduzir o exploit podem ser encontradas no seguinte link: https://github.com/noperator/CVE-2019-18935
- Neste exploit, o módulo deve carregar uma mixed mode .NET assembly DLL, que é carregada por meio da falha de desserialização. O upload do arquivo requer conhecimento das chaves criptográficas usadas pelo RAU. Os valores padrão usados ​​por este módulo estão relacionados com o CVE-2017-11317, que, após um patch, torna essas chaves aleatórias. Também é necessário saber a versão do Telerik UI ASP.NET que está a ser executada. Metasploit: https://www.rapid7.com/db/modules/exploit/windows/http/telerik_rau_deserialization/


## Ataques

- No inicio de junho a Australia sofreu um grande volume de ataques destinados a sistemas com versões do Telerik vulnerável ao CVE-2019-18935 que permite execução de código remota.
source: 
https://www.kroll.com/en-ca/insights/publications/cyber/monitor/telerik-vulnerability-surge-web-compromise-cryptomining-attacks 
https://www.cyber.gov.au/sites/default/files/2020-03/ACSC-Advisory-2020-004-Targeting-of-Telerik-CVE-2019-18935.pdf
- Dois dos webservers de produção da Telerik Web UI for ASP.NET foram infetados com um Bitcoin miner software através da vulnerabilidade do CVE-2019-18935. source: https://www.baco.sk/posts/xmrig-blue-mockingbird/

