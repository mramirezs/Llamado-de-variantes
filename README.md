# Llamado-de-variantes

### 1. **¿Qué diferencias debería buscar?**
- Las variantes siempre se definen en relación con una secuencia de referencia. 
- En este caso, se compara el genoma del brote de Ébola de 2014 con el de 1976.

---

### 2. **Preparación del Genoma de Referencia**
Se automatizan los pasos con un script:
1. Se define el número de acceso del genoma de referencia: `AF086833`.
2. Se descarga la secuencia con `efetch` y se convierte en formato FASTA usando `seqret`.
3. Se indexa la secuencia para alineaciones (`bwa index`) y visualización (`samtools faidx`).

**Comandos clave:**
```bash
ACC=AF086833
mkdir -p refs
REF=refs/$ACC.fa
efetch -db nuccore -format fasta -id $ACC | seqret -filter -sid $ACC > $REF
bwa index $REF
samtools faidx $REF
```

---

### 3. **Obtención de Datos de Secuenciación**
Los datos se obtienen del SRA (Sequence Read Archive) usando `fastq-dump`:
```bash
SRR=SRR1553500
fastq-dump -X 100000 --split-files $SRR
```
Esto genera dos archivos FASTQ (`_1.fastq` y `_2.fastq`) correspondientes a las lecturas pareadas.

---

### 4. **Alineación de Datos de Secuenciación**
Se utiliza `bwa mem` para alinear las lecturas contra la referencia:
1. Se añaden etiquetas (`@RG`) necesarias para herramientas como GATK.
2. El resultado es un archivo BAM ordenado con `samtools sort`.

**Comandos clave:**
```bash
BAM=$SRR.bam
R1=${SRR}_1.fastq
R2=${SRR}_2.fastq
TAG="@RG\tID:$SRR\tSM:$SRR\tLB:$SRR"
bwa mem -R $TAG $REF $R1 $R2 | samtools sort > $BAM
samtools index $BAM
```

---

### 5. **Llamado a Variantes con BCFtools**
Se utiliza `bcftools` para identificar variantes:
1. `mpileup` calcula probabilidades de genotipo.
2. `call` identifica las variantes y genera un archivo VCF.

**Comando clave:**
```bash
bcftools mpileup -O v -f $REF $BAM | bcftools call --ploidy 1 -vm -O v > variants1.vcf
```

---

### 6. **Llamado a Variantes con FreeBayes**
Otra herramienta para llamado a variantes es `FreeBayes`, que analiza directamente el archivo BAM:
```bash
freebayes -f $REF $BAM > variants2.vcf
```

---

### 7. **¿Qué es GATK (Genome Analysis Toolkit)?**
- GATK es un conjunto de herramientas avanzado para análisis genómicos, ideal para organismos complejos como humanos y ratones.
- Aunque poderoso, puede ser complicado de usar debido a su curva de aprendizaje y requisitos.
