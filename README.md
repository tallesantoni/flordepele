# Flor de Pele — Gestão Farmasi (pacote atualizado)

Este pacote contém o código-fonte do app web e os arquivos de configuração de contas. Baixar o ZIP **não publica nem atualiza automaticamente** o site que já está no ar. Login, sincronização e controle de acessos só funcionam após configurar o Supabase e publicar esta versão em HTTPS.

## Arquivos principais

- `index.html`, `billing-access.js`, `config.js`, `manifest.webmanifest`, `sw.js`, `icon.svg`: app web instalável.
- `supabase/migrations/001_commercial_accounts.sql`: perfis, dados por usuário e regras de acesso.
- `supabase/migrations/002_access_requests.sql`: tabela de solicitações de acesso.
- `supabase/functions/admin-users/index.ts`: função do servidor para solicitações, gestão de contas e senha temporária aleatória.

## Antes de começar

Crie um projeto Supabase próprio. Nunca coloque senha do banco ou chave `service_role` no `config.js`, no navegador ou em mensagens. A confirmação de pagamento é feita manualmente por você; o app não processa cobranças. Faça backup antes de substituir uma versão em uso: os dados locais antigos não são importados automaticamente para o banco.

## Configurar o Supabase e a conta administradora

1. No Supabase, abra **SQL Editor** → **New query**. Copie todo o conteúdo de `supabase/migrations/001_commercial_accounts.sql`, cole e clique **Run**.
2. Abra outra consulta e execute `supabase/migrations/002_access_requests.sql`. Rode primeiro 001 e depois 002.
3. Abra **Authentication** → **Users** → **Add user**. Crie a conta de administradora com seu e-mail e uma senha inicial; confirme o e-mail, se o painel oferecer essa opção.
4. Copie o UUID dessa conta na lista de usuários. No SQL Editor, execute o comando abaixo substituindo o texto entre aspas pelo UUID:

```sql
update public.profiles
set role = 'admin', status = 'active'
where id = 'COLE-O-UUID-DA-ADMIN-AQUI';
```

5. No painel Supabase, copie a **Project URL** e a chave **publishable** (em projeto antigo, `anon public`). Edite `config.js` e preencha:

```js
window.SITE_BACKEND = {
  url: "https://SEU-PROJETO.supabase.co",
  key: "SUA_CHAVE_PUBLICAVEL"
};
```

Não use a chave `service_role` nesse arquivo.

## Publicar a função administrativa

A função precisa ser implantada no Supabase para que os pedidos e aprovações funcionem. Em um computador com Supabase CLI instalado, abra o terminal na pasta do projeto e execute:

```text
supabase login
supabase link --project-ref SEU_PROJECT_REF
supabase functions deploy admin-users --no-verify-jwt
```

O `project-ref` aparece no endereço do projeto, antes de `.supabase.co`, ou nas configurações do Supabase. A função valida a sessão e a permissão administrativa internamente. Credenciais secretas ficam apenas no servidor Supabase, nunca no app.

## Hospedar e testar

1. Publique os arquivos do app em hospedagem HTTPS. A pasta `supabase/functions` é implantada separadamente pelo comando acima.
2. No Supabase, em **Authentication** → **URL Configuration**, adicione o endereço HTTPS publicado às URLs permitidas.
3. Entre como administradora e teste com um segundo e-mail seu: solicite acesso, confira o pagamento fora do app e use **Confirmar pagamento e liberar**. Copie a senha temporária exibida naquele momento e envie em canal privado; ela é mostrada uma única vez.
4. Entre com a conta de teste; confira que os dados ficam separados e teste suspender o acesso.

Sem Supabase, função implantada e site publicado com o `config.js` configurado, login e gerenciamento online não funcionarão. Não use o modo local de demonstração para dados reais de clientes nem comercialize antes dos testes.

## Atualizar o site existente

Este ZIP é o código-fonte, não uma atualização automática do link antigo. Para trocar a versão no ar, publique os arquivos na hospedagem que atende o domínio atual ou faça um novo deploy. Teste primeiro em um endereço separado e faça backup. Antes de vender, revise também privacidade, suporte, recuperação de acesso e requisitos da LGPD.

O pacote não inclui envio automático de senha por e-mail, cobrança recorrente ou processamento/verificação de pagamentos.
