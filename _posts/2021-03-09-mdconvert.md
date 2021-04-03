---
title: "Markdown Dateien konvertieren"
categories:
  - Blog
tags:
  - Markdown
  - Markdownlint
  - VSCode
  - Visual Studio Code
  - pandoc
  - LaTeX
---

# Aus Markdown mach Word und PDF

Microsoft verwendet in seinen Github Repositories für die Lab Anleitungen das Markdown Format. Leider wird dabei wenig bis gar keine Rücksicht auf die korrekte Syntax genommen. Man braucht also einen guten Editor mit Syntaxprüfung für Markdown Dateien. Außerdem soll man den Teilnehmern der Kurse die Lab Anleitungen zur Verfügung stellen, damit sie nicht selbst auf die Github Seiten gehen müssen. Das könnte für zusätzliche Verwirrung bei den Teilnehmern sorgen - sagt Microsoft.  
Also braucht man noch eine Möglichkeit aus Markdown Word und PDF Dokumente zu erstellen.

Im Folgenden zeige ich, wie man mit Visual Studio Code, ein paar Extensions, pandoc und MiKTeX die Syntaxprüfung und Konvertierung durchführen kann.

## Voraussetzungen

- Visual Studio Code [download](https://code.visualstudio.com/)
- pandoc [download](https://pandoc.org/installing.html)
- MiKTex [download](https://miktex.org/download)

Visual Studio Code wird durch zwei Extensions zum Editor und Konverter:

- markdownlint
- vscode-pandoc

## Los geht 's

1. Als erstes instaliert man die benötigte Software

    - Visual Studio Code ist ein ausgezeichneter Editor und durch allerelei Extensions erweiterbar.

    - Über Pandoc schreibt sein Schöpfer [John MacFarlane](https://johnmacfarlane.net):  
    *"If you need to convert files from one markup format into another, pandoc is your swiss-army knife."*  
    Unter anderem wandelt pandoc aus markdown auch in docx und pdf, benötigt für letzteres aber Unterstützung von LaTeX.

    - MiKtex - ausgesprochen *mick-tech* - ist eine aktuelle TeX/LaTeX Implementierung, die sich auch für Windows 10 Systeme eignet.  

2. Nun kommen die Extensions für Visual Studio Code an die Reihe.

    - Markdownlint von David Anson  
    ![mardownlint.png]({{site.url}}{{site.baseurl}}/assets/images/markdownlint.png)  
    Linter werden von Programmierern verwendet, um den Quellcode auf programmatische und stilistische Fehler zu überprüfen. Das hilft sowohl, Fehler proaktiv zu beheben, bevor sie auftreten, als auch einen Standardstil durchzusetzen, um den Code besser lesbar zu machen und leichter pflegen zu können.

    - vscode-pandoc von Chris Chinchilla  
    ![vscode-pandoc.png]({{site.url}}{{site.baseurl}}/assets/images/vscode-pandoc.png)  
    Da pandoc ein Kommandozeilen Tool ist, das sehr viele Optionen kennt, kommt uns diese Extension sehr gelegen, um unkompliziert und schnell markdown in word, pdf oder html zu konvertieren.

   Beide Extensions lassen sich über ihre Einstellungen konfigurieren, um z.B. bei Markdownlint die Regeln an die eigenen Bedürfnisse anzupassen.

3. Die pandoc Extension anpassen.

    Um das gewünschte Ergebnis der Konvertierung zu bekommen muss man pandoc die passenden Optionen mitgeben. Hierzu ist ein Blick in die Dokumentation von pandoc sehr hilfreich. Für Word Dokumente möchte ich gerne das Syntax-Highlighting aktivieren. Dazu klickt man auf das Zahnradsymbol der Extension und trägt unter *Pandoc: Docx Opt String* z.B. ein:
![docx-settings.png]({{site.url}}{{site.baseurl}}/assets/images/docx-settings.png)  
Das liefert schon recht ansehnliche Word Dokumente

    Etwas komplizierter wird es für pdf. Pandoc erstellt zunächst ein LaTeX Dokument und daraus dann das pdf Dokument. Hier ist mein Eintrag in den Settings unter  
    *Pandoc: Pdf Opt String*

    ```
    --highlight-style zenburn --pdf-engine=lualatex -V colorlinks -V urlcolor=NavyBlue -V geometry:"top=2cm, bottom=1.5cm, left=2cm, right=2cm" -V block-headings -V fontsize:12pt -V mainfont:"Times New Roman" -V monofont:"Consolas"
    ```

    Zur Bedeutung der Optionen verweise ich wieder auf die Dokumetation von pandoc.

4. Benutzung

    Hat man im Markdown Dokument alle Syntaxfehler beseitigt, kann man wie folgt Word und pdf Dokumente erstellen:
    F1 Taste drücken, Pandoc Render anklicken, das gewünschte Format wählen. Fertig!

5. Weitere Tipps

    - Mit den Standardeinstellungen von pandoc bekommt man zwar schon ansehnliche Dokumente. Aber möchte man eine andere Formatierung des Word Dokuments erreichen, z.B. das Format für Tabellen oder andere Schriftarten, muss man eine Referenz Datei angeben, die die gewünschten Einstellungen enthält. Für eine Datei im Verzeichnis *d:\\pandoc* mit dem Namen *custom-reference.docx*  ändert sich dann der *Pandoc: Docx Opt String* zu:

      `--highlight-style zenburn --reference-doc=d:\pandoc\custom-reference.docx`

    - eine weitere Datei kann bei Formatierungs-Problemen mit pdfs helfen. Beispielsweise werden beim Syntax Highlighting keine automatischen Umbrüche am Zeilenende gemacht. Hier schafft eine Datei *head.tex* mit folgendem Inhalt Abhilfe:  

      ```
      \usepackage{fvextra}
      \DefineVerbatimEnvironment{Highlighting}{Verbatim}{breaklines,commandchars=\\\{\}}
      ```

      Diese wird im ohnenhin schon sehr langen *Pandoc: Pdf Opt String* mit *-H d:\\pandoc\\head.tex* angegeben.

    - im *Pandoc: Pdf Opt string* ist **--pdf-engine=lualatex** angegeben. LaTeX kann auch mit anderen engines (z.B. pdflatex oder xelatex) das pdf Dokument erstellen. Aber hier habe ich bei xelatex etwas merkwürdiges festgestellt: Die Bindestriche in powershell cmdlets werden so unbrauchbar, dass man einzelne Befehle oder Skripte nicht per copy und paste übernehmen kann. Powershell erkennt dann den Befehl nicht!
