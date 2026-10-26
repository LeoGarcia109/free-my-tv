# Free My TV - Samsung Tizen Edition & Guia Completo de Operações

Repositório customizado do **Free My TV** adaptado especificamente para **Samsung Smart TVs (Tizen OS 5.5 / 2020+)**, com foco em desativação de bloatware, bloqueio de anúncios via DNS, injeção de catálogo dinâmico via TMDB e suporte a ícones transparentes para o mercado brasileiro.

---

## 📺 Especificações do Hardware Alvo (Laboratório de Produção)

* **Modelo:** Samsung Smart TV Crystal UHD 75" (Série 8)
* **Código do Modelo:** `UN75TU8000GXZD`
* **Plataforma de Hardware:** `20_NIKEL_UHD_BASIC`
* **Arquitetura da CPU:** `armv7l` (32-bit ARM Cortex)
* **Kernel:** `Linux Samsung 4.1.10 #1 SMP PREEMPT`
* **Sistema Operacional:** Tizen 5.5.0 (Codinome: Next)
* **Versão de Software / Firmware:** `T-NKLUABC-2743.0` (Build `T-NKLUABC-REL-202607022002`)
* **Armazenamento Interno:** Partição `/opt` (`/dev/mmcblk0p21`) com 3.9 GB total (2.5 GB usados, 1.5 GB disponíveis)

---

## 🚀 O que este Projeto Realiza

1. **Substituição da Interface Pesada:** Substitui o Smart Hub de fábrica por uma interface moderna, rápida e limpa em HTML5/CSS/JavaScript a 60 fps, reduzindo o uso de memória RAM em cerca de **60%**.
2. **Catálogo Dinâmico de Filmes e Séries:** Integração direta com a API do TMDB (TheMovieDB), preenchendo as abas *Explorar*, *Filmes* e *Séries* com os títulos em alta no Brasil disponíveis nos serviços de streaming instalados na TV.
3. **Ícones Vetoriais Transparentes (Brasil):** Elimina os ícones quadrados com fundo azul sólido que a Samsung força para o **Disney+** e **HBO Max**, substituindo-os por logotipos transparentes (`.svg`) sobre o degradê arredondado da prateleira.
4. **Clima e Finanças em Tempo Real:** Widget com previsão do tempo e pôr do sol sincronizados com Ilhéus / Bahia (°C) e cotações de criptoativos e dólar (R$ BRL) via Binance.
5. **Silenciamento de Telemetria e Anúncios:** Estratégia combinada com **AdGuard Home** na rede local para cortar chamadas de adtech e reconhecimento de tela (ACR) sem risco de corrupção do kernel.

---

## 🛠️ Topologia de Rede & Descobertas Críticas

### 1. O Problema do Duplo NAT (Roteadores Mesh Intelbras Twibi)
* **Cenário:** A TV estava conectada no nó Wi-Fi do Twibi (`192.168.5.108`) e o Mac Studio conectado via cabo Ethernet no modem da Huawei (`192.168.100.120`).
* **Barreira:** Por padrão, o Twibi opera em modo **Roteador com NAT/Firewall ativo**, bloqueando conexões de fora entrando na porta `26101` da TV.
* **Solução:** No app *Wi-Fi Control Home*, alterar o modo de operação do Twibi de **Roteador** para **Ponto de Acesso (Bridge)**. Os nós Twibi tornam-se antenas transparentes (Layer 2) e a TV ganha IP direto na sub-rede do roteador principal (`192.168.100.x`).

### 2. O Vazamento de DNS Secundário
* Sistemas operacionais consultam DNS Primário e Secundário de forma concorrente.
* **Ajuste:** No servidor DHCP do roteador, definir **tanto o Primário quanto o Secundário** para o IP do servidor DNS local (`192.168.100.120`), eliminando o bypass para servidores públicos desfiltrados.

### 3. O Furo do IPv6
* A operadora (Claro/Net) distribuía servidores DNS IPv6 via RDNSS, furando o bloqueio IPv4.
* **Ajuste:** Na aba de configuração LAN IPv6 do ONT, definir a fonte de DNS como **Static** e apontar o DNS IPv6 para o endereço IPv6 fixo do Mac Studio.

---

## 📡 Ativação do Modo Desenvolvedor na TV

Em controles remotos minimalistas (OneRemote sem teclado numérico físico):
1. Abra o **Smart Hub** e entre na aba **Apps**.
2. Pressione o botão virtual **`123`** no controle para exibir o teclado numérico na tela.
3. Selecione com o direcional a sequência: **`1` → `2` → `3` → `4` → `5`**. *(Alternativa: conectar qualquer teclado USB padrão na traseira da TV e digitar 12345).*
4. No pop-up azul **Developer Mode**:
   - Mude para **ON**.
   - No campo **Host PC IP**, digite o IP do seu computador na rede local (ex: `192.168.100.120`). *(Este campo autoriza o seu computador a se conectar; não é o IP da TV).*
   - Clique em **OK**.
5. **Hard Reboot obrigatório:** Segure o botão Liga/Desliga por 5 segundos até o logo da Samsung reaparecer, ou tire da tomada por 10 segundos. A porta **`26101`** estará aberta.

---

## 💻 Ambiente de Build e Assinatura Oficial (macOS Apple Silicon)

Para compilar e assinar pacotes `.wgt` aceitos pelo Tizen 5.5 sem necessitar do instalador gráfico pesado:

1. **Instalar OpenJDK via Homebrew:**
   ```bash
   brew install openjdk@21
   ```

2. **Montar o Tizen CLI Tools em `~/.hermes/tools/tizen-studio`:**
   Extrair os binários oficiais do repositório Tizen (`download.tizen.org/sdk/tizenstudio/official/binary/`):
   - `new-common-cli`
   - `certificate-generator`
   - `sdk-utils`
   - `sdk-utils-core`

3. **Arquivo de Configuração (`sdk.info`):**
   ```ini
   TIZEN_SDK_INSTALLED_PATH=/Users/leogarcia/.hermes/tools/tizen-studio
   TIZEN_SDK_DATA_PATH=/Users/leogarcia/.hermes/tools/tizen-studio-data
   JDK_PATH=/opt/homebrew/Cellar/openjdk@21/21.0.12.1/libexec/openjdk.jdk
   ```

4. **Geração do Certificado Oficial de Autor e Perfil de Segurança:**
   ```bash
   tizen certificate -a FreeMyTV -p SUA_SENHA -f freemytv_author
   tizen security-profiles add -n FreeMyTV -a ~/.hermes/tools/tizen-studio-data/keystore/author/freemytv_author.p12 -p SUA_SENHA -A -f
   tizen cli-config "profiles.path=/Users/leogarcia/.hermes/tools/tizen-studio-data/profile/profiles.xml"
   ```

---

## 📦 Compilação, Assinatura e Instalação na TV

1. **Configurar `config.properties`:**
   ```properties
   tmdb_key=SUA_CHAVE_TMDB
   tmdb_region=BR
   latitude=-14.7991
   longitude=-39.0323
   city=Ilhéus
   language=pt
   currency=BRL
   temperature=C
   theme=auto
   ```

2. **Compilar e Assinar com o Perfil Oficial:**
   ```bash
   export TIZEN_STUDIO=~/.hermes/tools/tizen-studio
   python3 tizen/build.py --sign FreeMyTV
   ```

3. **Instalar na TV via SDB:**
   ```bash
   python3 tizen/install.py <IP_DA_TV>
   ```

4. **Definir Inicialização Automática na TV:**
   - Na TV: **Configurações → Geral → Recursos Inteligentes**.
   - Ative **"Executar automaticamente último aplicativo"** (*Autorun Last App*).
   - Desative **"Executar automaticamente o Smart Hub"**.

---

## 🔍 IDs de Aplicativos de Streaming no Brasil (Mapeamento de Ícones)

Arquivos modificados em `web/tv.js`:
* **Max (HBO Max):** App ID `5b8c3eb16b.BeamCTVDev` (Pacote `5b8c3eb16b`) → mapeado para `icons/max.svg` e título `Max`.
* **Disney+:** App ID `MCmYXNxgcu.DisneyPlus` (Pacote `MCmYXNxgcu`) → mapeado para `icons/disney.svg` e título `Disney+`.
* **Netflix:** `RN1MCdNq8t.Netflix` / `org.tizen.netflix-app` → `icons/netflix.svg`.
* **Prime Video:** `evKhCgZelL.AmazonIgnitionLauncher2` / `org.tizen.primevideo` → `icons/prime.png`.
* **Apple TV:** `com.samsung.tv.aria-video` → ícone nativo transparente.
* **YouTube:** `9Ur5IzDKqV.TizenYouTube` → `icons/youtube.png`.

---

## 🛡️ Serviços de Fundo Bloqueados pelo AdGuard Home

A auditoria de processos via SDB revelou os seguintes serviços consumindo RAM e CPU na TV:
* `adagent-service` (`com.samsung.tv.pisa-service`): Injeta banners no menu.
* `acr.service` (`/usr/bin/acr`): Automatic Content Recognition (reconhecimento de imagem em tela).
* `tvplus` (`com.samsung.tv.tvplus`): Samsung TV Plus em segundo plano.
* `samsung-analytics` (`com.samsung.tizen.samsung-analytics`): Telemetria contínua.

Com o AdGuard Home ativo como DNS primário da TV, todos os domínios desses serviços (`*.samsungqbe.com`, `ads.samsung.com`, `samsungcloudsolution.net`, `samsungacr.com`) resolvem para `0.0.0.0` instantaneamente, mantendo a TV rápida e sem anúncios.

---

## 👨‍👩‍👧 9. Guardião da Família & Quadro de Recados (Noticeboard)

Integração nativa local-first entre o Mac Studio e a TV Samsung para gerenciamento de recados e rotina de sono da Fernanda.

### Como Funciona:
1. **Servidor Local no Mac Studio:** O daemon `tv_notice_server.py` roda na porta `192.168.100.120:9120` gerenciado pelo LaunchAgent `com.leogarcia.tv-notice-server.plist`.
2. **Atualização na TV:** A cada 30 segundos, o Free My TV consulta a API e renderiza um banner de vidro translúcido (`#family-guard`) localizado a `734px` de altura, logo acima da prateleira de apps.
3. **Regras da Rotina da Fernanda:**
   - **Segunda a Sexta:** Exibe aviso de que TV durante a semana é restrita (*"Hoje é dia de brincar e estudar! TV liberada no final de semana 🧸"*).
   - **Sábado e Domingo (17h45 às 18h00):** Aviso de contagem regressiva (*"Faltam X minutinhos para desligar a TV e tomar banho! ⏳"*).
   - **Sábado e Domingo (Após as 18h00):** Encerramento da sessão (*"Fim da sessão de hoje! Hora da rotina da noite da Fernanda 🌙"*).
4. **Recado Prioritário do Léo:** Quando você envia um recado, ele assume prioridade máxima na tela com o ícone 💌.
   - Enviar recado: `~/.hermes/scripts/tv_notice.sh "Texto do recado"`
   - Limpar recado: `~/.hermes/scripts/tv_notice.sh --clear`
