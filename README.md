<div align="center"> 
  <img alt="TryHackMe" src="https://repository-images.githubusercontent.com/518509014/f7450454-158c-45e0-8b38-0c0ae4d7394c" width="300px" /> <h1> 🛡️ Triagem de Alertas SOC L1</h1> Este projeto documenta a execução prática do laboratório de Triagem de Alertas SOC no TryHackMe, contendo exemplos de um Verdadeiro Positivo (TP) e um Falso Positivo (FP).
</div>

<br>

## 📑 Objetivos:
- Compreender a estrutura, o ciclo de vida e a importância dos alertas em um Centro de Operações de Segurança (SOC);
- Explorar campos de alerta, gerenciamento de status (Aberto, Em Andamento, Fechado) e metodologias de classificação.

<br>

## 🛠️ Ficha Técnica e Contexto Operacional:
- Plataforma: TryHackMe.
- Analista: [@Luanacyberdef](https://tryhackme.com/p/Luanacyberdef).
- Ambiente: Simulador de Dashboard SOC.
- Metodologia: Investigação baseada em evidências e análise de alertas.
- Sala no TryHackMe: [SOC L1 Alert Triage](https://tryhackme.com/room/socl1alerttriage).

<br>

## 🖥️ Visão Geral do Dashboard:
Ao iniciar a operação, foram identificados 5 alertas e 3 estavam pendentes na fila de triagem.
<img width="1242" height="456" alt="image" src="https://github.com/user-attachments/assets/8ab95f4e-c062-41f0-81e7-d04fde1dcf4f" />

> Simulador de Dashboard SOC.

<br>

### 📐 Propriedades de Alerta:

|       Propriedade       | Descrição                                                                                                             | Exemplo                                                                                                                             |
| :---------------------: | :-------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
|   **Tempo de Alerta**   | Horário de criação do alerta. Geralmente alguns minutos após o evento real.                                           | - Hora de alerta: 21 de março, 15:35H <br> - Horário do evento: 21 de março, 15:32H                                                 |
|    **Nome de Alerta**   | Resumo baseado na regra de detecção que gerou o alerta.                                                               | - Local de Login Incomum <br> - E-mail Marcado como Phishing <br> - Windows RDP Bruteforce <br> - Potencial de Exfiltração de Dados |
| **Gravidade do Alerta** | Define a urgência do alerta, inicialmente determinada pelos engenheiros de detecção. Pode ser ajustada pelo analista. | 🟢 Baixo / Informativo <br> 🟡 Médio / Moderado <br> 🟠 Alto / Severo <br> 🔴 Crítico / Urgente                                     |
|   **Status de Alerta**  | Indica se o alerta está sendo tratado ou se a triagem foi concluída.                                                  | 🆕 Novo / Não designado <br> 🔄 Em andamento / Pendente <br> ✅ Fechado / Resolvido                                                  |
|  **Veredito de Alerta** | Classificação indicando se o alerta é uma ameaça real ou não.                                                         | 🔴 Verdadeiro Positivo (TP) <br> 🟢 Falso Positivo (FP)                                                                             |
|  **Analista Designado** | Analista responsável pela revisão do alerta.                                                                          | - Também chamado de “dono do alerta”                                                                                                |

<br>

## 🚨 Alerta: Potencial de Exfiltração de Dados
### 1. Triagem dos Artefatos:
- **IP de Origem:** 192.168.45.66 (Localização: UK04 / Sala de Reuniões).
- **Destino:** .zoom.us (Serviço de Videoconferência).
- **Volume de Dados:** 5,8 GB Enviados / 5,2 GB Recebidos.

### 2. Análise Técnica:
- **Análise de Destino:** O domínio pertence ao Zoom, um serviço homologado e amplamente utilizado para comunicação. Não há indícios de comunicação com IPs de C2 ou domínios maliciosos conhecidos.
- **Avaliação de Contexto:** O tráfego originou-se de uma Sala de Reuniões. Em um cenário de trabalho híbrido ou reuniões globais, o consumo de 5GB de dados é compatível com uma sessão de vídeo em HD de longa duração (Ex: uma manhã inteira de conferência).
- **Conclusão da Análise:** Não foram encontrados sinais de abuso do serviço para fins de exfiltração; o comportamento é condizente com a função do ativo (sala de conferência).

### ✅ Veredito: Falso Positivo (FP)
- O incidente ocorreu porque a atividade legítima de videoconferência ultrapassou o limite (threshold) de volume configurado no SIEM.

### 📋 Recomendações sugeridas:
1. **Ajuste de Threshold:** Refinar a regra de correlação para considerar o "Tipo de Ativo", permitindo limites de tráfego maiores para dispositivos de salas de conferência;
2. **Evitar Whitelisting Total:** Em vez de liberar o domínio por completo (o que poderia ser explorado por atacantes usando contas pessoais), o ideal é ajustar os limites de alerta baseados no uso esperado.

## 🚨 Alerta: Criação de Arquivos com Dupla Extensão
### 1. Triagem dos Artefatos:
- **Host:** `LPT-HR-009` (Laptop do setor de Recursos Humanos);
- **Usuário:** `S.Conway`;
- **Processo Origem:** `chrome.exe` (Vetor: Download via navegador);
- **Arquivo Alvo:** `cats2025.mp4.exe` (Executável disfarçado de mídia);
- **URL de Origem (MotW):** `https://freecatvideoshd.monster/cats2025.mp4.exe`;
- **Hash MD5:** `14d8486f3f63875ef93cfd240c5dc10b`.

### 2. Análise Técnica:
- **Técnica de Mascaramento:** O arquivo utiliza a extensão composta `.mp4.exe.` Em sistemas Windows com extensões ocultas, o usuário veria apenas `cats2025.mp4`, acreditando ser um vídeo inofensivo.
- **Análise do Mark of the Web (MotW):** O recurso Mark of the Web confirma que o arquivo veio de uma zona de internet externa. **A URL utiliza um TLD de baixa reputação** (`.monster`) e nome apelativo, características típicas de infraestrutura de Malware Delivery.
- **Vetor de Ataque:** O download foi realizado via [Chrome](https://www.google.com/intl/pt-BR/chrome/), sugerindo que o usuário clicou em um link malicioso, possivelmente vindo de um e-mail ou anúncio (Phishing/Malvertising).

### ✅ Veredito: Verdadeiro Positivo (TP)
- A combinação de uma dupla extensão executável disfarçada de mídia, baixada de um domínio não confiável para a pasta de Downloads, confirma a tentativa de infecção por malware.

## 📝 Plano de Resposta Sugerido:
1. **Isolamento de Host:** Isolar o host LPT-HR-009 da rede para evitar uma possível movimentação lateral do malware;
2. **Remediação:** Excluir o arquivo malicioso e realizar uma varredura completa (Full Scan) com a ferramenta de EDR/Antivírus corporativo no dispositivo;
3. **Bloqueio de IOCs:** Adicionar o domínio malicioso à Blacklist do filtro web e o Hash MD5 à lista de bloqueio do endpoint para prevenir novas infecções na infraestrutura;
4. **Educação do Usuário:** Orientar o colaborador sobre os riscos de downloads em sites não oficiais e como identificar extensões suspeitas.

## 📍 Notas de Estudo do Alerta 1 e 2:
- Fadiga de Alertas: Limites muito baixos para serviços de vídeo geram muitos Falsos Positivos, sobrecarregando a fila de triagem.
- Técnica Low and Slow: É importante lembrar que exfiltrações reais podem ocorrer em volumes pequenos (abaixo do limite de 5GB) para evitar detecção. Isso exige que o analista observe padrões de rede por períodos mais longos.
- Dupla Extensão: Técnica que explora a confiança do usuário.
    - Ex: cats2025.mp4.exe mostra apenas a parte .mp4 no Windows Explorer se a opção "Ocultar extensões de tipos de arquivos conhecidos" estiver ativa.
- Mark of the Web (MotW): É um recurso de segurança do Windows que "carimba" arquivos vindos da internet.
    - Se o usuário tentasse abrir o arquivo, o SmartScreen provavelmente emitiria um alerta de risco.
- Importância do Isolamento: Em um cenário real de SOC, o isolamento do host é a primeira ação de contenção para garantir que uma ameaça não se espalhe pela rede antes da limpeza.

---

## 🏛️ Créditos e Direitos Autorais:
> [!WARNING]
> **Nota:** Este projeto faz parte de estudos práticos na plataforma [TryHackMe](https://tryhackme.com/).
> Todos os direitos sobre laboratórios, marcas e infraestrutura pertencem à respectiva plataforma.

## 📜 Licença
> [!IMPORTANT]
> O conteúdo autoral deste repositório está licenciado sob a licença **Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**.
> Veja: 👉 [LICENSE](./LICENSE.md)

## 🤖 Uso de IA
> Parte deste conteúdo foi elaborada com apoio de ferramentas de IA, utilizadas como auxílio na organização e redação do texto, com revisão e validação integral pelo autor.

---

## 🔗 Compartilhe com a comunidade 🧡

Por favor, se esse conteúdo te ajudou, não esqueça de compartilhar 😁

[![GitHub Repo stars](https://img.shields.io/badge/share%20on-twitter-03A9F4?logo=twitter)](https://twitter.com/share?url=https://github.com/Luanacyberdef/Triagem-de-Alertas-SOC-L1) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-facebook-1976D2?logo=facebook)](https://www.facebook.com/sharer/sharer.php?u=https://github.com/Luanacyberdef/Triagem-de-Alertas-SOC-L1) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-linkedin-3949AB?logo=linkedin)](https://www.linkedin.com/shareArticle?url=https://github.com/Luanacyberdef/Triagem-de-Alertas-SOC-L1)
