# A Real CTF

## Enunciado

O desafio **A Real CTF** consiste em jogar um nível secreto que não está presente na build entregue ao jogador, mas está embutido no binário do verificador. Para resolver o desafio, é necessário descobrir a existência desse nível, extraí-lo do binário ELF e então jogá-lo corretamente para gerar um replay válido.

Endpoint do verificador remoto: https://chall.polygl0ts.ch:8533

<img width="1047" height="463" alt="image" src="https://github.com/user-attachments/assets/073b8a8a-e6b7-44ca-a2f6-664230039018" />

---

## Arquivos

- **Verifier**: Binário responsável por validar o replay e que contém o nível secreto embutido.

- **a_real_ctf_linux.zip** e **a_real_ctf_windows.zip**: Builds do jogo para Linux e Windows, respectivamente.  

---

## Ideia geral da solução

A solução segue os seguintes passos:

1. Descobrir que existe um nível secreto embutido no verificador.
2. Extrair o `game_config` diretamente do binário ELF do verificador.
3. Carregar o nível secreto na build do jogo.
4. Jogar o nível e gerar um replay válido.
5. Enviar o replay ao verificador remoto para obter a flag.

O jogo já vem com o **BepInEx** instalado, um mod loader que permite a execução de código C# customizado. A engenharia reversa do jogo pode ser feita analisando o arquivo `A_REAL_CTF_Data/Manager/Assembly-CSharp.dll` com ferramentas como o dnSpy. Apesar disso, o uso do BepInEx não é obrigatório: seria possível modificar diretamente arquivos do jogo ou até reimplementar a lógica e a física necessárias.

---

## Análise do verificador

A análise do arquivo `verifier` foi feita com **radare2**. Durante a inspeção, foi identificada a função:



