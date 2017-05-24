Per esportare in pdf usare pandoc. N.B: pandoc ha bisogno di un'installazione di LaTeX, si consiglia MikTeX.

`pandoc -s -N --from=markdown --to=latex --latex-engine=xelatex -H ./subtitle.tex -V subtitle="Vulnerabilità e messa in sicurezza dell'applicazione MyUniversity" -V documentclass="scrreprt" ./report.md -o ./report.pdf"`