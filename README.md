# Clear Jupyter Notebook

The python script [clear_ipynb.py](clear_ipynb.py) can be used to clear (and strip solutions from) a Jupyter Notebook.

## Usage

### Examples

- Clear all the outputs (+ execution counters + scrolled status), remove empty code cells (but not markdown ones), remove non standard global metadata (like `colab` for example). The result is saved in the original `example.ipynb` file without making a backup.
  ```bash
  ./clear_ipynb.py -i --no-backup example.ipynb
  ```

- Clear "all" : the outputs (+ execution counters + scrolled status), remove the empty cells (code and markdown ones), remove non standard global metadata. All solutions are also removed from `homework_with_solutions.ipynb` and all cells are protected from removal, those not containing solutions are also protected from editing. The result is saved to the `homework.ipynb` file in the parent directory.
  ```bash
  ./clear_ipynb.py -a -o ../homework.ipynb homework_with_solutions.ipynb
  ```

### Help

```bash
./clear_ipynb.py --help
usage: clear_ipynb.py [-h] [-o out.ipynb] [-i] [-a] [-s] [--no-scrolled]
                      [--no-outputs] [--no-execution-count] [--keep-metadata]
                      [--no-empty-code] [--empty-markdown]
                      [--no-protect-cells] [--no-backup]
                      in.ipynb

Clear a Jupyter notebook:
- clear cell outputs [except if --no-outputs]
- clear cell metadata.scrolled [except if --no-scrolled]
- clear cell execution counts [except if --no-execution-count]
- clear metadate to keep only the standard parts [except if --keep-metadata]
- remove empty code cells [except if --no-empty-code]
- remove empty markdown cells [if --empty-markdown]
- strip solutions (parts between comments)  [if --strip-solutions or -s]
    - in this case protect cells [except if --no-protect-cells] against deletion and edition

positional arguments:
  in.ipynb              the file name of jupyer notebook to clear

optional arguments:
  -h, --help            show this help message and exit
  -o out.ipynb, --out-file out.ipynb
                        save to the indicated file
  -i, --in-place        edit file in-place, save backup as <filename>_bak.ipynb
  -a, --all             maximal cleaning = -s --empty-markdown
  -s, --strip-solutions
                        strip the solutions, and save to indicated file
  --no-scrolled         do not clear metadata.scrolled
  --no-outputs          do not clear outputs
  --no-execution-count  do not clear execution counts
  --keep-metadata       do not clear the global metadata
  --no-empty-code       do not remove empty code cells
  --empty-markdown      remove empty markdown cells
  --no-protect-cells    do not protect cells from deletion/edition
  --no-backup           do not backup the original [valid only with -i]
```

## How it works

### Structure of Jupyter files

The Jupyter `.ipynb` files are `json` files with [the following structure](https://nbformat.readthedocs.io/en/latest/format_description.html):

```txt
{
 "cells": [ // a typical cell
  {
   "cell_type": "(markdown|code)",
   "metadata": {...},
   "outputs": [...], // only for code cells
   "source": ["line1","line2",...] // python, markdown, ... source code
  },
  ... // more cells
 ],
 "metadata": {
    "kernelspec": {...},
    "language_info": {...},
    ...
  },
 "nbformat": X,
 "nbformat_minor": Y
}
```

### How the script works

The script use `json.loads` to parse the `.ipynb`, then depending on the flags reset some values, and save back to `json` using `json.dump`.

### How the solutions are stripped

Any code placed between `# --- sol(ution)...` comments is considered as solution:

```python
# --- solution ---

... here is the code to remove ...

# --- solution ---
```

and any markdown between `<!--- sol(ution) --->` is considered also as solution:

```markdown
<!--- solution --->

**Solution:** ... here is the text to remove ...

<!--- solution --->
```

When the flag `-s` (`--strip-solutions`) or `-a` (`--all`) is set, all the solutions are removed from the notebook.

### How and why the cells are protected

In the resulting notebook without solutions, by default, all the cells are protected from deletion (by inserting `"deletable": false` in the cells `metadata`), and all cells that do not contain solutions (where the students are not supposed to write) are protected from editing (by inserting `"editable": false` in the cells `metadata`).

In this way the students can edit only the cells that are supposed to be edited. The students can also add new editable/deletable cells.

This protection is not "secure" because the student can easily remove this restriction. This protection is here to prevent accidents and to facilitate the verification of the work by the teacher (as explained below).

### Facilitate verification of student work

When the teacher has to check the student's work, he can highlight the cells that are editable, i.e. the cells where the student (is supposed to) have done his work.

To accomplish this just execute the following `javascript` code in the page containing the notebook:

```js
for (const cell of Jupyter.notebook.get_cells()) {
  if(! ("editable" in cell.metadata) || cell.metadata.editable) {
    cell.element.addClass("solution")
  }
}

function addStyle(str) {
    var node = document.createElement('style');
    node.innerHTML = str;
    document.body.appendChild(node);
}

addStyle(".solution {background: #ffefff;}")
```
This code can be saved in a bookmark to facilitate its execution:

```js
javascript:(function(){for(const e of Jupyter.notebook.get_cells())"editable"in e.metadata&&!e.metadata.editable||e.element.addClass("solution");function addStyle(e){var t=document.createElement("style");t.innerHTML=e,document.body.appendChild(t)}addStyle(".solution {background: #ffefff;}");})();
```

