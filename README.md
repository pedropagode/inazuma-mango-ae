# Inazuma's Mango GUI — Win32 + Direct2D + DirectComposition (fundação)

Migração de PyQt6 (`gui.py`, 6537 linhas) para C++20 puro, sem framework de
UI — Win32 nativo + Direct2D + DirectComposition, conforme decidido no
chat. Esta é a **fundação + 3 controles**, não o app completo.

## O que está aqui

| Arquivo | Substitui (Python) | Status |
|---|---|---|
| `Core/D2DRenderer.h` | back-end implícito do QPainter/QWidget | Completo — device D3D11, swap chain de composição, target D2D |
| `Core/Window.h` | `MacroGUI(QWidget)` (gui.py:2354+) | Fundação: HWND frameless, DPI, loop de mouse/frame, drag pelo fundo |
| `Core/ThemeManager.h` | `_theme_palette()` (gui.py:452-555) | Cores portadas 1:1, sem QString — usa `D2D1_COLOR_F` |
| `Controls/Control.h` | papel base de `QWidget` | Completo |
| `Controls/Label.h` | `QLabel` | Completo |
| `Controls/Button.h` | `QPushButton` + QSS `#BtnStart/#BtnStop/#BtnPS` | Completo — gradiente, hover, pressed |
| `Controls/ToggleSwitch.h` | `_AlienToggle` (gui.py:753-853) | **Alta fidelidade** — mesma geometria, cores, easing, 8 passos |
| `App/main.cpp` | topo + bootstrap de `gui.py` | Cria janela + 1 Label + 1 Button + 1 Toggle de demonstração |

## O que NÃO está aqui (ainda)

- Corpo real da MainWindow (painéis start/stop/stats/settings) — 6537
  linhas do Python, a maior parte do trabalho ainda não portada.
- Fonte "League Spartan" real (usando Segoe UI como fallback por ora).
- Opacidade de fade via `IDCompositionVisual::SetOpacity` — o código tem
  o "esqueleto" da animação de fade mas o setter de opacidade real ainda
  não está ligado (marcado com TODO em `Window::SetLayeredWindowAttributesIfNeeded`).
- Sliders, ScrollView, TextBox, ComboBox, DragReorderList, SlidingPanel.
- Ícone da janela, imagem de fundo, blur/Mica.

## Por que parei aqui

Não tenho MSVC/Direct2D neste ambiente para compilar e validar contra
erros reais — só posso revisar linha a linha contra a API documentada.
Prefiro entregar uma base pequena e você compilar/testar antes de eu
empilhar mais código em cima de algo que pode ter um erro de assinatura
de função ou include faltando.

## Como compilar

Requer Visual Studio 2022 com "Desktop development with C++" (conforme
Etapa 1 do guia que você já tem). **Não precisa de vcpkg nem Qt** — só
Windows SDK, que já vem com o VS.

```cmd
cmake -B build -S . -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release
```

O executável fica em `build/Release/InazumaMangoGUI.exe`.

## O que você deve ver rodando

Uma janela frameless 460x670, título "Inazuma's Mango" no canto superior
esquerdo, um botão "Configurações" (gradiente laranja do tema padrão,
resposta a hover/click — veja o Output do Visual Studio para a mensagem de
debug ao clicar), e um toggle switch roxo/verde com label "OFF"/"ON" ao
lado que anima suavemente ao clicar. Arrastar a janela clicando em
qualquer área vazia do fundo deve funcionar.

## Próximo passo recomendado

Me manda os erros de compilação (se houver) primeiro — é comum ter
pequenos ajustes de include/assinatura em código Direct2D não testado.
Depois disso, os próximos controles a portar em ordem de prioridade real
do app: **Slider**, **campo de texto validado (moeda/gems)**, e o corpo do
painel de settings (SlidingPanel + abas). Peça um de cada vez.
