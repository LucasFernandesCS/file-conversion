# Conversor distribuído de TXT para PDF

Implementação da atividade **APS 02** da disciplina CIN0143 – Introdução aos Sistemas Distribuídos e Redes de Computadores (UFPE), referente à **Equipe 05**. O sistema converte arquivos de texto em PDF através de uma arquitetura cliente-servidor com sockets TCP.

O servidor atende vários clientes simultaneamente. Cada conexão mantém o estado da própria transferência e usa um arquivo temporário com identificador único, para que conversões concorrentes não compartilhem dados. O limite de entrada é 64 MiB.

## Requisitos

- Node.js 22 ou superior;
- npm.

## Instalação e execução

```bash
git clone https://github.com/luismiguuel/file-conversion.git
cd file-conversion
npm install
```

Em um terminal, inicie o servidor:

```bash
npm run server
```

O servidor escuta em `127.0.0.1:5000`. Em outro terminal, na raiz do projeto, inicie o cliente:

```bash
npm run client
```

Digite o nome de um arquivo `.txt` que esteja na raiz do projeto. O arquivo de exemplo incluído é `teste.txt`. O PDF será salvo em `convertidos/` com o mesmo nome-base.

Para compilar o TypeScript:

```bash
npm run build
```

## Protocolo TCP

1. O cliente envia `CONVERT txt pdf <nome>\n`.
2. O servidor valida o comando e responde `ACCEPT\n` ou `REJECT <motivo>\n`.
3. Após `ACCEPT`, o cliente envia o tamanho do TXT como inteiro sem sinal de 4 bytes em *big-endian*, seguido dos bytes do arquivo.
4. O servidor converte o conteúdo usando PDFKit e devolve o tamanho do PDF no mesmo formato de 4 bytes, seguido dos bytes do documento.
5. O cliente recebe os bytes, salva o PDF e informa o caminho.

Os comprimentos explícitos delimitam cada arquivo sem depender dos limites entre pacotes TCP. O servidor associa cada conversão à conexão que a solicitou e remove o PDF temporário após o envio.

## Estrutura

```text
src/
├── client/
│   ├── client.ts       # Interface de linha de comando
│   └── connect.ts      # Protocolo TCP e gravação do PDF
└── server/
    ├── servidor.ts     # Aceitação de conexões e controle da transferência
    └── conversor.ts    # Conversão TXT para PDF com PDFKit
teste.txt               # Entrada de exemplo
```

As pastas `temp/` e `convertidos/` são criadas durante a execução quando necessário.
