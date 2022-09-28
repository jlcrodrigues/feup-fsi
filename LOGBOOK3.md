
# Trabalho realizado na Semana #3

## CVE-2012-0158

### Identificação

- O CVE-2012-0158 num buffer overflow na biblioteca MSCOMCTL.OCX do Windows.
- A vulnerabilidade é mais comum em certas versões de Plataformas Microsoft como Office, SQL Server, Visual Basic,...
- O ataque ocorre quando um utilizador visita uma página web, abre um documento do Microsoft Office ou um ficheiro RTF e tem uma versão vulnerável instalada numa das plataformas acima referidas.
- Um ataque bem sucedido garante direitos de utilizador autenticado ao atacante. 

### Catalogação

- O nível de gravidade da vulerabilidade, em CVSS 2.0, é 9.3 (Crítico).
- Este tipo de ataque já foi detectado por investigadores de segurança, nomeadamente em agências governamentais na Europa e na Ásia, entre outros.
- Apesar de ter sido corrigido em Abril 2012, a vulnerabilidade ainda existe em certas versões de software.
- Manteve-se bem sucedido durante 4 anos. Isto porque  muitos utilizadores demoraram a atualizar os seus produtos para versões mais recentes.

### Exploit

- item1
- item2
- item3
- item4

### Ataques

- A vulnerabilidade foi usada em 2013 para atacar entidades governamentais da Europa e da Ásia. O ataque utilizou emails de phishing para introduzir documentos maliciosos que permitiam obter acesso backdoor aos sistemas vulneráveis.
- Permitiu também criar um malware usado em ataques na Ásia, denominado “KeyBoy”. Através da partilha de documentos de word maliciosos, permitia ganhar acesso backdoor aos computadores com versões do Office vulneráveis.
- O método foi utlizado para criar uma ameaça denominada Trojan.APT.BaneChant, que permitia monitorizar os cliques do rato nos computadores infetados. 
- O CVE-2012-0158 foi explorado para criar documentos de word maliciosos que permitiram lançar uma campanha de espionagem a agências governamentais do Paquistão.

## CTF Sanity check

O desafio foi completo copiando a flag que se encontra nas regras.
