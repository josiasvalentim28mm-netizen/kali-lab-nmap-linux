# Kali Lab: Reconhecimento com Nmap e Permissões em Linux

Projeto prático do meu laboratório caseiro de cibersegurança, feito num portátil dedicado com Kali Linux. Todos os testes foram feitos **apenas na minha própria rede doméstica e em aparelhos meus**.

## Preparação do hardware: análise e troca do SSD

Antes de instalar o sistema, preparei uma máquina antiga para ser o laboratório dedicado.

1. **Análise da saúde dos SSD:** usei o CrystalDiskInfo para verificar o estado dos discos disponíveis.
2. **Escolha do disco:** selecionei um SSD com **96% de saúde**.
3. **Troca:** instalei esse SSD na minha máquina antiga.

## Instalação do Kali Linux

Com o SSD novo instalado, prossegui com a instalação do Kali Linux na máquina antiga.

| Item | Detalhe |
| --- | --- |
| Máquina | Dell Latitude (portátil antigo) |
| Disco de destino | SSD Kingston de 240 GB |
| Meio de instalação | Pen USB de 64 GB com a imagem oficial do Kali |
| Ambiente de trabalho | Xfce (leve, ideal para hardware mais antigo) |

**Cuidado que tive:** confirmar sempre o disco de destino pelo tamanho, para não apagar a pen de instalação.

## Objetivos

- Fazer reconhecimento de uma rede doméstica com o Nmap
- Identificar portas abertas e serviços em diferentes aparelhos
- Perceber como os aparelhos IoT reagem a scans repetidos
- Compreender utilizadores, privilégios e permissões de ficheiros no Linux

## Ambiente

| Item | Detalhe |
| --- | --- |
| Máquina | Portátil dedicado, com SSD novo |
| Sistema | Kali Linux |
| Ferramentas | Nmap, terminal Bash |
| Rede | Rede doméstica com router do operador |

## Parte 1: Descoberta de aparelhos na rede

```bash
sudo nmap -sn 192.168.X.0/24
```

- `-sn` faz apenas a descoberta de aparelhos (ping scan), sem verificar portas.
- `/24` percorre os 256 endereços da rede.

**Resultado:** 4 aparelhos encontrados em cerca de 2 segundos: o router, uma smart TV, o meu PC principal e o próprio portátil com o Kali.

**Erro que aprendi a corrigir:** escrevi primeiro `nmap-sn`, tudo junto, e o terminal não reconheceu o comando. É preciso um espaço entre o comando e as opções.

## Parte 2: Portas abertas no router

```bash
sudo nmap 192.168.X.1
```

| Porta | Estado | Serviço | Observação |
| --- | --- | --- | --- |
| 53 | open | DNS | Normal num router |
| 80 | open | HTTP | Página de configuração sem encriptação |
| 443 | open | HTTPS | Página de configuração encriptada |
| 23 | filtered | Telnet | Serviço antigo e inseguro, mas bloqueado: bom sinal |
| 8022 | filtered | desconhecido | Bloqueado pela firewall |

**Conclusão:** o router expõe apenas os serviços de que precisa.

## Parte 3: Portas abertas na smart TV

```bash
sudo nmap -F -T4 192.168.X.TV
```

- `-F` (fast) verifica só as 100 portas mais comuns.
- `-T4` acelera o scan, adequado numa rede doméstica.

| Porta | Uso provável |
| --- | --- |
| 7000 | AirPlay |
| 8008 | Google Cast / Chromecast |
| 8009 | Google Cast (encriptado) |
| 8443 | Porta web segura usada por apps |
| 9000 | Serviços internos da TV |

**Conclusão:** a TV tem muito mais portas abertas do que o router. É típico dos aparelhos IoT, que trazem muitos serviços ativos de fábrica, e é por isso que são alvos frequentes.

**Lição:** `-t4` em minúsculas deu erro. No Linux, maiúsculas e minúsculas contam, até nas opções.

## Parte 4: Deteção de versões e defesa da TV

```bash
sudo nmap -sV -F -T4 192.168.X.TV
```

- `-sV` tenta identificar o programa e a versão em cada porta.

**Resultado:** desta vez, todas as 100 portas apareceram como **filtered**.

**Interpretação:** depois de vários pedidos seguidos, a TV começou a ignorar o portátil, ou estava em modo de espera, com os serviços de rede desligados.

**Próximo teste:** fazer o scan só às portas já conhecidas, para gerar menos tráfego:

```bash
sudo nmap -sV -p 7000,8008,8009,8443,9000 192.168.X.TV
```

## Parte 5: Utilizadores e privilégios no Linux

```bash
whoami        # mostra: <utilizador>
sudo whoami   # mostra: root
```

- Sem `sudo`, trabalho como utilizador normal.
- Com `sudo`, ganho poderes de administrador (root) só durante aquele comando.
- Boa prática: não trabalhar sempre como root, porque um erro pode afetar o sistema inteiro.

## Parte 6: Permissões de ficheiros

```bash
touch teste.txt
ls -l
```

Exemplo de permissões de uma pasta: `drwxr-xr-x`

| Parte | Significado |
| --- | --- |
| `d` | É uma pasta (`-` seria um ficheiro) |
| `rwx` | Dono: ler, escrever, executar/entrar |
| `r-x` | Grupo: ler e entrar, sem alterar |
| `r-x` | Outros: ler e entrar, sem alterar |

### Alterar permissões com chmod

`chmod` vem de *change mode*.

| Comando | Efeito | Resultado |
| --- | --- | --- |
| `chmod o-r teste.txt` | Tira a leitura aos outros | `-rw-rw----` |
| `chmod o+r teste.txt` | Devolve a leitura aos outros | `-rw-rw-r--` |
| `chmod g-w teste.txt` | Tira a escrita ao grupo | `-rw-r--r--` |
| `chmod g+w teste.txt` | Devolve a escrita ao grupo | `-rw-rw-r--` |

**Regra:** `u` = dono, `g` = grupo, `o` = outros; `+` dá e `-` tira; `r`, `w`, `x` = ler, escrever, executar.

## O que aprendi

- O reconhecimento é a primeira fase tanto de um atacante como de um analista de segurança.
- Aparelhos IoT expõem muitos serviços e podem reagir a scans repetidos.
- Pequenos detalhes de sintaxe, como espaços e maiúsculas, fazem diferença no terminal.
- O controlo de permissões é uma base essencial da segurança em Linux.

## Ética

Todos os scans foram feitos exclusivamente na minha rede e nos meus aparelhos. Fazer scans a redes de terceiros sem autorização pode ser ilegal.
