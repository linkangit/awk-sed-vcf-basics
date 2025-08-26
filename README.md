# Beginner's Guide to VCF Parsing with awk and sed

## What is a VCF file?

Think of a VCF (Variant Call Format) file as a fancy spreadsheet that scientists use to store information about genetic differences. When researchers compare your DNA to a reference genome, they find spots where your DNA is different - these are called "variants." The VCF file is basically a list of all these differences, along with information about how confident we are that each difference is real.

## Sample VCF File

Let's work with this example throughout our tutorial. Don't worry if it looks scary at first - we'll break it down step by step!

```
##fileformat=VCFv4.2
##INFO=<ID=DP,Number=1,Type=Integer,Description="Total Depth">
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	Sample1	Sample2
chr1	1000	.	A	G	99.9	PASS	DP=50	GT	0/1	1/1
chr1	2000	rs123	C	T	85.3	PASS	DP=30	GT	0/0	0/1
chr2	3000	.	G	A	45.2	PASS	DP=20	GT	1/1	0/0
chr2	4000	rs456	T	C	12.1	LOWQUAL	DP=15	GT	0/1	0/1
chrX	5000	.	A	T	78.9	PASS	DP=40	GT	0/1	0/0
```

Save this as `example.vcf` and you can follow along with every example!

## Understanding VCF Structure (Don't Panic!)

Here's what's going on in this file:
- Lines starting with `##` are like footnotes - they explain what everything means
- The line starting with `#CHROM` is like a column header in Excel
- The actual data rows contain the variants (genetic differences) we found
- Each column tells us something different: which chromosome, where on that chromosome, what the change was, how confident we are, etc.

Think of it like this: "Hey, on chromosome 1 at position 1000, we found that instead of the usual 'A', this person has a 'G'. We're pretty confident about this (quality score 99.9)."

## Let's Start Simple with awk

awk is like having a really smart assistant who can read through your file line by line and pick out exactly what you want. Let's see what it can do!

### 1. "Just Show Me the Data!" (Skip All Those Header Lines)

```bash
awk '!/^#/' example.vcf
```

**What you'll see:**
```
chr1	1000	.	A	G	99.9	PASS	DP=50	GT	0/1	1/1
chr1	2000	rs123	C	T	85.3	PASS	DP=30	GT	0/0	0/1
chr2	3000	.	G	A	45.2	PASS	DP=20	GT	1/1	0/0
chr2	4000	rs456	T	C	12.1	LOWQUAL	DP=15	GT	0/1	0/1
chrX	5000	.	A	T	78.9	PASS	DP=40	GT	0/1	0/0
```

**What's happening here?**
- `!/^#/` is awk's way of saying "show me lines that do NOT start with #"
- The `!` symbol means "not" (like saying "I don't want this")
- `^#` means "starts with #"
- So we're telling awk: "I don't want to see any of those messy header lines, just give me the good stuff!"
- This is super handy because VCF files are cluttered with lots of technical headers that you usually don't care about
- Now you can actually see your variants without all the noise!

### 2. "I Only Care About Certain Columns"

```bash
awk '!/^#/ {print $1, $2, $4, $5}' example.vcf
```

**What you'll see:**
```
chr1 1000 A G
chr1 2000 C T
chr2 3000 G A
chr2 4000 T C
chrX 5000 A T
```

**What's happening here?**
- `$1` = chromosome (the first column)
- `$2` = position (the second column) 
- `$4` = reference allele (what's normally there)
- `$5` = alternative allele (what we found instead)
- The `print` command is like saying "only show me these specific pieces of information"
- Think of it like highlighting only the columns you care about in a spreadsheet
- This is perfect when you just want the essential info: "Where is this variant and what changed?"

### 3. "Make It Easy to Read!"

```bash
awk '!/^#/ {print "Chr:", $1, "Pos:", $2, "Change:", $4">"$5}' example.vcf
```

**What you'll see:**
```
Chr: chr1 Pos: 1000 Change: A>G
Chr: chr1 Pos: 2000 Change: C>T
Chr: chr2 Pos: 3000 Change: G>A
Chr: chr2 Pos: 4000 Change: T>C
Chr: chrX Pos: 5000 Change: A>T
```

**What's happening here?**
- We're adding friendly labels like "Chr:" and "Pos:" to make everything crystal clear
- `$4">"$5` is a neat trick that puts the reference and alternative alleles together with a ">" arrow
- The quotes around text create labels that appear exactly as written
- Now instead of cryptic columns, you get readable sentences like "Chr: chr1 Pos: 1000 Change: A>G"
- It's like turning a boring spreadsheet into a human-friendly report!
- Much easier to understand what's actually happening at each position

## Time to Get Picky - Filtering Your Data

Now we're going to teach awk to be selective. Think of it like having a bouncer at a club who only lets in the variants that meet your criteria!

### 4. "Only Show Me the Good Stuff!" (High Quality Variants)

```bash
awk '!/^#/ && $6 > 50' example.vcf
```

**What you'll see:**
```
chr1	1000	.	A	G	99.9	PASS	DP=50	GT	0/1	1/1
chr1	2000	rs123	C	T	85.3	PASS	DP=30	GT	0/0	0/1
chrX	5000	.	A	T	78.9	PASS	DP=40	GT	0/1	0/0
```

**What's happening here?**
- `$6` is the quality score column (think of it like a confidence score)
- `> 50` means we only want scores greater than 50
- `&&` means "AND" - so both conditions must be true (no headers AND good quality)
- Quality scores are like grades - higher numbers mean we're more confident the variant is real
- This filters out the sketchy, low-quality variants that might just be sequencing errors
- You're basically saying "I only want the variants you're really sure about!"

### 5. "Show Me Only the Ones That Passed the Test"

```bash
awk '!/^#/ && $7 == "PASS"' example.vcf
```

**What you'll see:**
```
chr1	1000	.	A	G	99.9	PASS	DP=50	GT	0/1	1/1
chr1	2000	rs123	C	T	85.3	PASS	DP=30	GT	0/0	0/1
chr2	3000	.	G	A	45.2	PASS	DP=20	GT	1/1	0/0
chrX	5000	.	A	T	78.9	PASS	DP=40	GT	0/1	0/0
```

**What's happening here?**
- `$7` is the FILTER column - think of it like a quality control stamp
- `== "PASS"` means we want exact matches to the word "PASS"
- The double equals `==` is like asking "is this exactly equal to..."
- When a variant says "PASS", it means it survived all the quality checks
- Variants that say "LOWQUAL" or other things failed some test and might be false alarms
- It's like only accepting packages that have passed inspection - you want the reliable stuff!

### 6. Show Variants from Specific Chromosome

```bash
awk '!/^#/ && $1 == "chr1"' example.vcf
```

**Output:**
```
chr1	1000	.	A	G	99.9	PASS	DP=50	GT	0/1	1/1
chr1	2000	rs123	C	T	85.3	PASS	DP=30	GT	0/0	0/1
```

**Explanation:**
- `$1 == "chr1"` means the first column (chromosome) must equal "chr1"
- This filters the data to show only variants from chromosome 1
- You can change "chr1" to any chromosome you're interested in
- This is useful when studying specific chromosomes or genes
- You could use "chr2", "chrX", "chrY", etc.

### 7. Combine Multiple Filters

```bash
awk '!/^#/ && $1 == "chr1" && $6 > 80' example.vcf
```

**Output:**
```
chr1	1000	.	A	G	99.9	PASS	DP=50	GT	0/1	1/1
chr1	2000	rs123	C	T	85.3	PASS	DP=30	GT	0/0	0/1
```

**Explanation:** 
- This combines two conditions with `&&` (AND)
- First condition: `$1 == "chr1"` (must be chromosome 1)
- Second condition: `$6 > 80` (quality must be greater than 80)
- Both conditions must be true for a variant to be shown
- This gives you high-quality variants from chromosome 1 only
- You can combine as many conditions as needed

## Let's Count Some Stuff!

Sometimes you don't want to see all the data - you just want to know "how many?" awk is great at being your personal accountant.

### 8. "How Many Variants Do I Have?"

```bash
awk '!/^#/ {count++} END {print "Total variants:", count}' example.vcf
```

**What you'll see:**
```
Total variants: 5
```

**What's happening here?**
- `count++` is like having a tally counter that clicks up by 1 for each variant
- The `++` symbol is shorthand for "add 1 to this number"
- `END` means "after you're done reading the whole file, do this"
- The variable `count` starts at 0 automatically (awk is helpful like that)
- This gives you a quick answer to "how big is my dataset?"
- Super useful for getting a bird's eye view of your data before diving in!

### 9. Count Variants Per Chromosome

```bash
awk '!/^#/ {count[$1]++} END {for (chr in count) print chr, count[chr]}' example.vcf
```

**Output:**
```
chr1 2
chr2 2
chrX 1
```

**Explanation:**
- `count[$1]++` creates a counter for each chromosome name
- `$1` is the chromosome column, so `count["chr1"]++` counts chr1 variants
- This creates an array where each chromosome is a key
- `for (chr in count)` loops through each chromosome in our array
- `print chr, count[chr]` shows each chromosome and its count
- This tells you how variants are distributed across chromosomes

### 10. Count High vs Low Quality Variants

```bash
awk '!/^#/ {
    if ($6 > 50) high++; 
    else low++
} 
END {
    print "High quality:", high
    print "Low quality:", low
}' example.vcf
```

**Output:**
```
High quality: 3
Low quality: 2
```

**Explanation:**
- `if ($6 > 50)` checks if quality score is greater than 50
- If true, `high++` increases the high-quality counter
- If false, `else low++` increases the low-quality counter
- The `if-else` structure handles both cases
- At the end, we print both counters
- This gives you a quality breakdown of your variants
- Helps you understand the overall data quality

## Now Let's Meet sed - The Text Editor

sed is like having a really fast copy editor who can make changes to your file. While awk is great at analyzing, sed is your go-to for cleaning up and modifying text.

### 1. Remove Header Lines

```bash
sed '/^#/d' example.vcf
```

**Output:**
```
chr1	1000	.	A	G	99.9	PASS	DP=50	GT	0/1	1/1
chr1	2000	rs123	C	T	85.3	PASS	DP=30	GT	0/0	0/1
chr2	3000	.	G	A	45.2	PASS	DP=20	GT	1/1	0/0
chr2	4000	rs456	T	C	12.1	LOWQUAL	DP=15	GT	0/1	0/1
chrX	5000	.	A	T	78.9	PASS	DP=40	GT	0/1	0/0
```

**Explanation:** 
- `/^#/` matches lines that start with # (the ^ symbol means "starts with")
- `d` means "delete" those lines
- This removes all header and metadata lines from the VCF file
- The result is only the actual variant data
- Different from awk approach - sed physically removes the lines
- Useful when you want to clean up the file permanently

### 2. Replace "chr" with "chromosome"

```bash
sed 's/^chr/chromosome/' example.vcf
```

**Output:**
```
##fileformat=VCFv4.2
##INFO=<ID=DP,Number=1,Type=Integer,Description="Total Depth">
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	Sample1	Sample2
chromosome1	1000	.	A	G	99.9	PASS	DP=50	GT	0/1	1/1
chromosome1	2000	rs123	C	T	85.3	PASS	DP=30	GT	0/0	0/1
chromosome2	3000	.	G	A	45.2	PASS	DP=20	GT	1/1	0/0
chromosome2	4000	rs456	T	C	12.1	LOWQUAL	DP=15	GT	0/1	0/1
chromosomeX	5000	.	A	T	78.9	PASS	DP=40	GT	0/1	0/0
```

**Explanation:**
- `s/` starts a substitution command (s = substitute)
- `^chr` matches "chr" at the beginning of a line (^ = start of line)
- `/chromosome/` is what we replace it with
- The final `/` ends the command
- This changes "chr1" to "chromosome1", "chr2" to "chromosome2", etc.
- Only replaces at the start of lines, so it won't change "chr" in the middle of text
- Useful for standardizing chromosome naming conventions

### 3. Keep Only Header Line and Data (Remove Metadata)

```bash
sed '/^##/d' example.vcf
```

**Output:**
```
#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	Sample1	Sample2
chr1	1000	.	A	G	99.9	PASS	DP=50	GT	0/1	1/1
chr1	2000	rs123	C	T	85.3	PASS	DP=30	GT	0/0	0/1
chr2	3000	.	G	A	45.2	PASS	DP=20	GT	1/1	0/0
chr2	4000	rs456	T	C	12.1	LOWQUAL	DP=15	GT	0/1	0/1
chrX	5000	.	A	T	78.9	PASS	DP=40	GT	0/1	0/0
```

**Explanation:**
- `/^##/` matches lines that start with ## (double hash)
- `d` deletes these lines
- This removes metadata lines (like ##fileformat, ##INFO, etc.)
- But keeps the column header line that starts with just one # 
- The column header (#CHROM POS ID...) is important for understanding data
- Metadata lines are often not needed for basic analysis
- Results in a cleaner file with just the essential information

## Practical Examples

### 4. Create a Simple Report

```bash
awk '
/^#CHROM/ {print "=== VCF ANALYSIS REPORT ===\n"}
!/^#/ {
    total++
    if ($7 == "PASS") passed++
    if ($6 > 50) highqual++
} 
END {
    print "Total variants found:", total
    print "Variants that passed filters:", passed  
    print "High quality variants (>50):", highqual
}' example.vcf
```

**Output:**
```
=== VCF ANALYSIS REPORT ===

Total variants found: 5
Variants that passed filters: 4
High quality variants (>50): 3
```

**Explanation:**
- `/^#CHROM/` matches the column header line and prints a report title
- `\n` creates a blank line for better formatting
- For each data line: `total++` counts all variants
- `if ($7 == "PASS")` counts variants that passed quality filters
- `if ($6 > 50)` counts high-quality variants (score > 50)
- `END` section runs after all lines are processed
- Creates a summary report showing key statistics
- Great for getting a quick overview of your VCF file quality

### 5. Extract Only SNPs (Single Letter Changes)

```bash
awk '!/^#/ && length($4)==1 && length($5)==1 {
    print $1, $2, $4">"$5, "Quality:"$6
}' example.vcf
```

**Output:**
```
chr1 1000 A>G Quality:99.9
chr1 2000 C>T Quality:85.3
chr2 3000 G>A Quality:45.2
chr2 4000 T>C Quality:12.1
chrX 5000 A>T Quality:78.9
```

**Explanation:**
- `length($4)==1` checks if the reference allele is exactly 1 character long
- `length($5)==1` checks if the alternative allele is exactly 1 character long
- `&&` means both conditions must be true
- This filters for SNPs (Single Nucleotide Polymorphisms) - single letter changes
- Excludes insertions, deletions, or complex variants
- `$4">"$5` shows the change (e.g., A>G means A changed to G)
- SNPs are the most common type of genetic variation
- Useful when you only want to study point mutations

### 6. Find Variants in a Position Range

```bash
awk '!/^#/ && $1=="chr1" && $2>=1000 && $2<=2500 {
    print "Found variant at position", $2, ":", $4">"$5
}' example.vcf
```

**Output:**
```
Found variant at position 1000 : A>G
Found variant at position 2000 : C>T
```

**Explanation:**
- `$1=="chr1"` restricts search to chromosome 1
- `$2>=1000` means position must be 1000 or higher
- `$2<=2500` means position must be 2500 or lower
- `&&` connects all conditions with AND logic
- This finds variants in a specific genomic region (chr1:1000-2500)
- Position numbers refer to base pair locations on the chromosome
- Useful for studying variants in specific genes or regions of interest
- You can change the chromosome and position range for different areas

## Combining awk and sed

### 7. Clean File and Then Analyze

```bash
sed '/^##/d' example.vcf | awk '
/^#/ {print "Starting analysis..."; next}
{
    print "Chromosome", $1, "has variant", $4">"$5, "at position", $2
}'
```

**Output:**
```
Starting analysis...
Chromosome chr1 has variant A>G at position 1000
Chromosome chr1 has variant C>T at position 2000
Chromosome chr2 has variant G>A at position 3000
Chromosome chr2 has variant T>C at position 4000
Chromosome chrX has variant A>T at position 5000
```

**Explanation:**
- The `|` symbol pipes output from sed to awk (connects the commands)
- `sed '/^##/d'` first removes metadata lines (lines starting with ##)
- The cleaned output goes to awk for analysis
- `/^#/ {print "Starting analysis..."; next}` handles the remaining header
- `next` skips to the next line without processing further
- For data lines, creates readable descriptions of each variant
- This shows how to chain sed and awk commands together
- First clean/prepare data with sed, then analyze with awk

### 8. Create a CSV File

```bash
awk '
BEGIN {print "Chromosome,Position,Reference,Alternative,Quality"}
!/^#/ {print $1","$2","$4","$5","$6}
' example.vcf
```

**Output:**
```
Chromosome,Position,Reference,Alternative,Quality
chr1,1000,A,G,99.9
chr1,2000,C,T,85.3
chr2,3000,G,A,45.2
chr2,4000,T,C,12.1
chrX,5000,A,T,78.9
```

**Explanation:**
- `BEGIN` runs before processing any lines - creates the CSV header
- Commas separate the column names in the header row
- `!/^#/` processes only data lines (skips VCF headers)
- `$1","$2","$4","$5","$6` joins columns with commas
- Creates proper CSV (Comma-Separated Values) format
- CSV files can be opened in Excel, Google Sheets, or other programs
- This converts VCF data to a more universally readable format
- You can add `> output.csv` to save to a file instead of printing to screen

## Practice Time - Try These Yourself!

Here are some challenges to test your new skills. Don't worry if you get stuck - that's how we learn!

1. **"Count variants per chromosome that are high quality"** (Hint: combine chromosome filtering with quality > 40)
2. **"Show me everything from chromosome 2"** (Should be pretty straightforward now!)
3. **"Make it say 'Position 1000 on chr1 changed from A to G'"** (Think about how to build a sentence)
4. **"Tell me how many passed vs failed quality filters"** (This one's a bit tricky - you'll need to count two different things)
5. **"Remove 'chr' from all chromosome names"** (sed time!)

## Quick Cheat Sheet (Bookmark This!)

**awk magic spells:**
- `!/^#/` = "ignore those pesky headers"
- `$1, $2, $3` = "give me column 1, 2, 3"
- `$NF` = "give me the last column"  
- `&&` = "AND - both things must be true"
- `||` = "OR - either thing can be true"
- `==` = "exactly equals"
- `>`, `<` = "greater than, less than"

**sed magic spells:**
- `s/old/new/` = "change 'old' to 'new'"
- `/pattern/d` = "delete lines that match this pattern"
- `/pattern/p` = "print lines that match this pattern"

**Your VCF columns (memorize these!):**
1. Chromosome (`$1`) - where the variant is
2. Position (`$2`) - exact location on that chromosome
3. ID (`$3`) - variant name (often just a dot)
4. Reference allele (`$4`) - what's normally there
5. Alternative allele (`$5`) - what we found instead
6. Quality score (`$6`) - how confident we are (higher = better)
7. Filter status (`$7`) - did it pass quality control?
8. Info (`$8`) - extra technical details
9. Format (`$9`) - how to read the sample data
10+ Sample data (`$10`, `$11`, etc.) - actual genotype for each person

## Final thoughts

Congrats! You now know how to wrangle VCF files like a pro :) Start with simple commands and gradually combine them to create more powerful analysis scripts. Remember, the best way to learn is by doing it yourself, grab a VCF file and start experimenting. Happy varaint calling!
