Per esportare in pdf usare pandoc. N.B: pandoc ha bisogno di un'installazione di LaTeX, si consiglia MikTeX.

`pandoc --number-sections --from=markdown "/path/to/markdown/file.md" --to=latex -o "/path/to/output/file.pdf"`