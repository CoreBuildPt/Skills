# CoreBuild Skills

Coleção de skills para Claude Code / AI coding agents, curada pela CoreBuild.

## Skills incluídas

### `ui-skills/` — by [ibelick](https://github.com/ibelick/ui-skills)
Pequenas skills focadas em fundamentos UI/UX:
- **baseline-ui** — design system fundamentals
- **fixing-accessibility** — a11y audit/fixes
- **fixing-metadata** — meta tags / SEO
- **fixing-motion-performance** — animation perf

### `ui-ux-pro-max-skill/` — by [nextlevelbuilder](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)
Antigravity Kit: toolkit AI-powered de design intelligence.
Search databases para UI styles, color palettes, font pairings, chart types, UX guidelines.

**Uso principal:**
```bash
python3 src/ui-ux-pro-max/scripts/search.py "<query>" --domain style
```

**Domínios:** product · style · typography · color · landing · chart · ux

**Stacks:** html-tailwind · react · nextjs · astro · vue · nuxtjs · svelte · swiftui · react-native · flutter · shadcn · jetpack-compose

## Como usar estas skills num projeto

1. Clonar este repo no teu home folder: `git clone https://github.com/CoreBuildPt/Skills ~/.claude/skills-corebuild`
2. Claude Code detecta skills automaticamente em `~/.claude/skills/`
3. Copiar pastas relevantes: `cp -r ~/.claude/skills-corebuild/ui-ux-pro-max-skill ~/.claude/skills/`

## Roadmap

- [ ] Adicionar skill custom baseada em [impeccable.style](https://impeccable.style) (design references)
- [ ] Skill para Portuguese/Spanish SEO (BLABLABOAT specific)
- [ ] Skill para Next.js + Prisma + Postgres boilerplate

## Licença
Cada sub-skill mantém sua licença original (ver LICENSE files respetivos).
