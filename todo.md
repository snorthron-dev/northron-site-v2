# Northron Security – TODO

## Banco de Dados e Backend
- [x] Schema do banco: tabelas orcamentos, page_views, eventos, lista_espera
- [x] Migrations executadas com pnpm db:push
- [x] Helpers de DB: criarOrcamento, listarOrcamentos, contarOrcamentosPorStatus
- [x] Helpers de DB: registrarPageView, obterEstatisticasPageViews, obterTotalPageViews
- [x] Helpers de DB: registrarEvento, obterEstatisticasEventos
- [x] Helpers de DB: inscreverListaEspera, listarListaEspera
- [x] Helpers de DB: listarUsuarios, atualizarRoleUsuario
- [x] Router tRPC: analytics (registrarPageView, registrarEvento)
- [x] Router tRPC: orcamentos (criar, buscar)
- [x] Router tRPC: listaEspera (inscrever)
- [x] Router tRPC: admin (stats, listarOrcamentos, atualizarStatus, listarListaEspera, listarUsuarios)
- [x] Notificação ao dono quando novo orçamento é criado

## CSS e Design
- [x] Tema preto profundo + dourado elite (CSS variables)
- [x] @theme inline para Tailwind 4 (border-border, bg-background, etc.)
- [x] Fontes premium: Playfair Display + Montserrat via Google Fonts
- [x] Classes utilitárias: btn-gold, btn-outline-gold, card-premium, section-title, divider-gold
- [x] Animações: fadeIn, slideInLeft, slideInRight, fadeInUp
- [x] Componente AnimatedSection com Intersection Observer

## Componentes
- [x] Navbar premium com logo, links e CTA dourado
- [x] Footer com links, contato e redes sociais
- [x] WhatsAppButton flutuante contextual por página
- [x] AnimatedSection para fade-in ao scroll

## Páginas Públicas (11 páginas)
- [x] Home – hero cinematográfico, stats, serviços, depoimentos, CTA
- [x] Sobre – história honesta da empresa, missão, valores
- [x] Metodologia – 6 etapas detalhadas do processo
- [x] Treinamentos – 3 níveis (básico, intermediário, avançado)
- [x] Guarda e Proteção – guarda patrimonial e proteção pessoal
- [x] Planos – consultoria avulsa e pacotes com preços
- [x] Resultados – casos reais com antes/depois e depoimentos
- [x] FAQ – 10+ perguntas frequentes com acordeão
- [x] Orçamento – formulário completo com opt-in para promoções
- [x] Promoções – lista de espera para receber ofertas
- [x] Contato – informações de contato e formulário rápido

## Painel Administrativo (4 páginas)
- [x] AdminDashboard – stats gerais, links rápidos
- [x] AdminOrcamentos – listagem com filtros e atualização de status
- [x] AdminLeads – lista de espera de inscritos
- [x] AdminUsuarios – gerenciamento de usuários

## Testes
- [x] Testes de conexão com banco de dados
- [x] Testes de CRUD de orçamentos
- [x] Testes de lista de espera
- [x] Testes de analytics (page views)
- [x] Todos os 10 testes passando

## Pendente / Futuro
- [ ] Fotos reais do instrutor e dos cães (aguardando envio pelo cliente)
- [ ] Vídeos de treinamento (aguardando envio pelo cliente)
- [ ] Depoimentos reais de clientes (após primeiros atendimentos)
- [ ] SEO: meta tags, Open Graph, sitemap

## Correções
- [x] Corrigir tela de acesso restrito no admin para mostrar botão de login
- [x] Corrigir redirecionamento pós-login para voltar ao /admin após autenticar

## Sistema de Autenticação Própria
- [ ] Instalar bcryptjs para hash de senhas
- [ ] Schema: tabela admin_users com username, passwordHash, nomeCompleto, telefone, cargo, fotoUrl, role, ativo, primeiroAcesso
- [ ] Roles: admin, adestrador, recepcao
- [ ] Migração do banco (pnpm db:push)
- [ ] Helpers de DB: criarAdminUser, buscarPorUsername, atualizarSenha, atualizarPerfil, listarAdminUsers
- [ ] Endpoint tRPC: adminAuth.login (username + senha)
- [ ] Endpoint tRPC: adminAuth.logout
- [ ] Endpoint tRPC: adminAuth.trocarSenha (apenas própria senha, primeiro acesso)
- [ ] Endpoint tRPC: adminAuth.me (retorna usuário logado)
- [ ] Endpoint tRPC: admin.criarUsuario (apenas admin)
- [ ] Endpoint tRPC: admin.editarUsuario (apenas admin: nome, telefone, cargo, foto)
- [ ] Endpoint tRPC: admin.redefinirSenha (apenas admin redefine senha de outro usuário)
- [ ] Endpoint tRPC: admin.ativarDesativar (apenas admin)
- [x] Tela de login /admin/login com username e senha
- [ ] Fluxo de primeiro acesso: redireciona para /admin/redefinir-senha
- [ ] Contexto de auth próprio (useAdminAuth hook)
- [ ] Proteção de rotas admin por role
- [ ] Painel de usuários: listar, criar, editar, foto de perfil, redefinir senha
- [ ] Upload de foto de perfil para S3
- [ ] Criar usuário admin inicial: thiago.northron.adm / Adonai.01
- [ ] Corrigir redirecionamento pós-login: após login bem-sucedido, ir para /admin
