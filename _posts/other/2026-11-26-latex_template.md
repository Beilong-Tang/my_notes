---
title: Latex presentation template
tag: Latex
---

{% raw %}
```latex
% Beamer version of 11-05-2026.pptx
%   style mimics ../10-30-2026.pptx: white background, Times font, red (#FF0000)
%   titles, black bullets (• / ➢), slide number bottom-right, no other decoration.
\documentclass[aspectratio=169,11pt,t]{beamer}  % t = top-aligned content, like the PPT

% ---------- Packages ----------
\usepackage[T1]{fontenc}
\usepackage{newtxtext,newtxmath}   % Times New Roman look
\usefonttheme{serif}
\usepackage{amsmath}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{array}
\usepackage{colortbl}
\usepackage{pifont}                % \ding{226} = ➢
\usepackage{tikz}
\usetikzlibrary{arrows.meta,positioning,calc,shapes.geometric,decorations.pathmorphing,decorations.pathreplacing}
\graphicspath{{figures/}}

% ---------- Colors ----------
\definecolor{pptred}{HTML}{FF0000}
\definecolor{pptgray}{HTML}{8C8C8C}
\definecolor{pptorange}{HTML}{F8931D}
\definecolor{pptblue}{HTML}{2998E3}

\setbeamercolor{normal text}{fg=black,bg=white}
\setbeamercolor{structure}{fg=black}
\setbeamercolor{frametitle}{fg=pptred,bg=}
\setbeamercolor{title}{fg=pptred}
\setbeamercolor{section title}{fg=pptred}
\setbeamercolor{item}{fg=black}
\setbeamercolor{block title}{fg=pptred,bg=}
\setbeamercolor{block body}{fg=black,bg=}
\setbeamercolor{alerted text}{fg=pptred}

% ---------- Layout ----------
\setbeamertemplate{navigation symbols}{}
\setbeamersize{text margin left=0.55cm, text margin right=0.55cm}

% Frame title: left-aligned, large, red (PPT: 44pt on a 13.33in slide)
\setbeamerfont{frametitle}{size=\fontsize{20}{24}\selectfont}
\setbeamertemplate{frametitle}{%
  \vspace*{0.35cm}%
  \usebeamerfont{frametitle}\usebeamercolor[fg]{frametitle}%
  \insertframetitle\par\vspace*{0.1cm}%
}

% Body text sizes (PPT: 24 / 22 / 20 pt)
\setbeamerfont{itemize/enumerate body}{size=\fontsize{11.5}{14}\selectfont}
\setbeamerfont{itemize/enumerate subbody}{size=\fontsize{10.5}{13}\selectfont}
\setbeamerfont{itemize/enumerate subsubbody}{size=\fontsize{9.5}{12}\selectfont}

% Bullets: • then ➢
\setbeamertemplate{itemize item}{\textbullet}
\setbeamertemplate{itemize subitem}{\ding{226}}
\setbeamertemplate{itemize subsubitem}{\ding{226}}
\setbeamertemplate{enumerate items}[default]

% Footer: only the slide number, bottom-right, gray
\setbeamertemplate{footline}{%
  \hfill{\fontsize{7.5}{9}\selectfont\color{pptgray}\insertframenumber}%
  \hspace*{0.6cm}\vspace*{0.25cm}%
}

% Blocks: plain (no shading)
\setbeamertemplate{blocks}[default]

% Title page: centered red title, then date, then author
\setbeamertemplate{title page}{%
  \vfill
  \centering
  {\fontsize{28}{32}\selectfont\usebeamercolor[fg]{title}\inserttitle\par}
  \vspace{0.25cm}
  {\fontsize{11.5}{14}\selectfont\color{black}\insertdate\par}
  \vspace{0.15cm}
  {\fontsize{11.5}{14}\selectfont\color{black}\insertauthor\par}
  \vfill
}

% Section header slide (PPT "Section Header" layout): big red title, left
\newcommand{\sectionslide}[1]{%
  \begin{frame}[plain]
    \vspace*{2.2cm}
    {\fontsize{28}{32}\selectfont\color{pptred}#1\par}
    \vfill
    \hfill{\fontsize{7.5}{9}\selectfont\color{pptgray}\insertframenumber}\hspace*{0.05cm}
  \end{frame}}

% ---------- Math shortcuts ----------
\newcommand{\Sph}{\mathbb{S}}
\newcommand{\R}{\mathbb{R}}
\newcommand{\E}{\mathbb{E}}
\newcommand{\Exp}{\mathrm{Exp}}
\newcommand{\Log}{\mathrm{Log}}
\newcommand{\Beta}{\mathrm{Beta}}
\newcommand{\SLERP}{\mathrm{SLERP}}
\newcommand{\red}[1]{\textcolor{pptred}{#1}}

% ---------- Algorithm box (Mijatović et al., Algorithm 1) ----------
% \bmalg{<style for lines 2-3>}{<style for lines 4-6>}
\newcommand{\algl}[2]{\makebox[1.3em][r]{\footnotesize #1:}\ #2\par}
\newcommand{\bmalg}[2]{%
  \begin{minipage}{0.92\textwidth}
    \rule{\linewidth}{0.8pt}\par\vspace{1pt}
    \textbf{Algorithm 1} Simulating from the transition density $p_y(\cdot,\tau)$ of spherical Brownian motion\par
    \vspace{-2pt}\rule{\linewidth}{0.4pt}\par\vspace{2pt}
    \algl{1}{Start point $y \in \Sph^{d-1}$, time $\tau > 0$}
    {#1\algl{2}{Simulate $M \sim A_\infty^{d-1}(\tau)$}
    \algl{3}{Simulate $X \sim \Beta\!\left(\frac{d-1}{2},\, \frac{d-1}{2} + M\right)$}}
    {#2\algl{4}{Simulate $Y$ uniformly on $\Sph^{d-2}$}
    \algl{5}{Set $O(y) = I - 2uu^\top$, where $u = \dfrac{e_d - y}{\|e_d - y\|}$}
    \algl{6}{\textbf{return} $O(y)\,\big(2\sqrt{X(1-X)}\,Y^\top,\ 1-2X\big)^\top$}}
    \vspace{-2pt}\rule{\linewidth}{0.8pt}
  \end{minipage}}

% ---------- Title info ----------
\title{Progress Report:\\ Sphere diffusion}
\author{Beilong Tang}
\date{11/05/2026}

\begin{document}

% =====================================================================
\begin{frame}
  \titlepage
\end{frame}

% =====================================================================
\begin{frame}{Template}
  \begin{itemize}
    \item template
    \item math here $x^2 = 2x$
  \end{itemize}
\end{frame}

\end{document}
```
{% endraw %}

