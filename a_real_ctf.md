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

<img width="1687" height="474" alt="image" src="https://github.com/user-attachments/assets/409a59a5-5c7f-4165-928e-a6b8eff8408a" />

Para achar o endereço da função:

<img width="1630" height="204" alt="image" src="https://github.com/user-attachments/assets/5dbc0cc1-144b-498b-addb-6cb261297d84" />

Essa função é relativamente complexa, mas precisamos analisar os seguintes trechos:

<img width="1017" height="199" alt="image" src="https://github.com/user-attachments/assets/13477e8b-1dcc-42d6-989a-38b6d6434c7b" />

Essa são as informações que precisamos para saber onde extrair, onde o verificador lê dados do próprio arquivo, e o tamanho da leitura:

OFFSET = 0x2e1f6
LENGTH = 0x374

Essa função está carregando o `game_config` (que contém o nível secreto do CTF) a partir dessa posições, ou seja, o verificador está _auto-contido_, carregando dados escondidos dentro dele.

A partir dessas informações dentro do verifier, vamos rodar o seguinte script para extrair as informações que estão nesse trecho:

```python
# offset e tamanho encontrados na função load()
offset = 0x2e1f6
length = 0x374

# lê do binário
with open("../verifier", "rb") as f:
    f.seek(offset)
    data = f.read(length)

# salva o game_config
with open("game_config.dat", "wb") as out:
    out.write(data)

print(f"Extracted {len(data)} bytes into game_config.dat")
```

Esse script criou o ``game_config.dat`` com base no arquivo ``verifier``.

<img width="1008" height="111" alt="image" src="https://github.com/user-attachments/assets/c592d269-838f-423e-8669-58936647dbd9" />

Agora vamos montar o ambiente, precisamos do game_config.dat e o SolveMod.dll:

<img width="1030" height="93" alt="image" src="https://github.com/user-attachments/assets/565575da-626f-4602-9b40-568caeac8864" />

O plugin `SolveMod.dll` faz:

- interceptar funções do jogo
- inserir código customizado
- alterar física
- carregar níveis alternativos
- substituir arquivos internos
- automatizar ações

(``.dll`` desenvolvido por terceiro)

Uma vez que colocamos os arquivos dentro da pasta do jogo, vamos desbloquear a nova missão secreta.

<img width="1050" height="549" alt="image" src="https://github.com/user-attachments/assets/0c9d4e40-a366-4fe5-ab83-eb89a9c24399" />

Ao jogar, finalizamos a missão para pegar o arquivo que o jogo gera da fase e salvando como ``win.replay``.

<img width="1045" height="544" alt="image" src="https://github.com/user-attachments/assets/1cb9bc5d-7a1a-44f8-b1eb-0795626e856a" />

Por fim, com o win.replay em mão, vamos rodar o submit.py.

```python
import requests

with open("win.replay", "rb") as f:  # lê o replay
    data = bytearray(f.read())

url = "https://chall.polygl0ts.ch:8533"  # verificador remoto
payload = bytes(data)  # dados a enviar

response = requests.post(url, data=payload)  # envia o replay
print(response.content)  # resposta (flag ou erro)
```

Ele envia o arquivo `win.replay` para o verificador oficial do CTF (https://chall.polygl0ts.ch:8533) e imprime a resposta.

<img width="1030" height="117" alt="image" src="https://github.com/user-attachments/assets/e9b6aadc-4643-45e5-8aff-0cdc002b756f" />




