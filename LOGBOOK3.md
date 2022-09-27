# Trabalho realizado na Semana #3

# Heap-based buffer overflow in Sudo (CVE-2021-3156)

## Identificação:

- Esta vulnerabilidade trata-se de um heap-based overflow que permite utilizadores locais utilizarem permissões sudo.

- Esta exploit afeta todos os sistemas operativos que usam o pacote Sudo. Isto inclui a maioria dos sistemas baseados em UNIX.

- Foi introduzido em julho de 2011 (commit 8255ed69) e afeta as versões legacy 1.8.2 a 1.8.31p2 e as versões estáveis de 1.9.0 a 1.9.5p1 do pacote Sudo.


## Catalogação

- Esta vulnerabilidade foi descoberta pelo Qualys Research Labs no dia 26/01/2021. 

- Não existia uma bug-bounty relacionada com esta falha, e a sua eventual correção, veio por meio de uma contribuição independente.

- Este CVE tem um nivel CVSS de 7.2, o que o coloca no patamar de risco alto. 

## Exploit

- É possivel a elevação de previlégios para "root" através do comando "sudoedit -s", seguido de um argumento que termina com um unico caracter de backslash.

- Existe um módulo de metasploit que permite a automação de ataques com esta fraqueza. 

# Ataques

- Existem diversos exploits que foram criados para documentação. Contudo, não encontramos indicios de utilização para ataques.

- No entanto, todos os sistemas que utilizem estas versões compremetidas, são vulneraveis a que qualquer utilizador tenha permissões de "root".

- Um utilizador root, pode fazer transmutações ao sistema operativo incluindo mudanças que podem causar danos maliciosos irreversíveis. 





