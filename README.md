![header](https://capsule-render.vercel.app/api?type=waving&color=gradient&height=200&section=header&text=SpikyClip🧷&desc=Vikesh%20Ajith&descAlignY=70&fontSize=40)

# Hi there, I'm **Vikesh Ajith** (*SpikyClip*)

I'm a data engineer with a background in bioinformatics. I currently work for
the [Next-Generation Precision Medicine Program
(NGPMP)](https://www.hudson.org.au/research-program/vpcc-next-generation-precision-medicine/)
at the Hudson Institute's Centre for Cancer Research (CCR).

Fewer than 1 in 5 children with cancer are found to have actionable mutations
that are targetable with existing drug therapies. To improve these odds, our
program has developed an extensive collection of paediatric cancer cell line
models which then undergo a comprehensive set of functional genomic screens to
identify novel drivers of low-survival paediatric cancers. High-thoroughput drug
screens are then used to identify potential treatments that precisely target
these novel mutations. Published data is then made available through the
[Childhood Cancer Model Atlas
(CCMA)](https://lookerstudio.google.com/u/1/reporting/933d232a-b8dd-4801-8fe5-5e9214d2722b/page/p_9i9wsspiad).

### My Contributions

Our program produces terabytes of data that has to be stored, processed,
cleaned, and annotated before being disseminated to researchers for downstream
analysis. My role as data engineer is to effectively manage the above data
lifecycle so that researchers can spend more time on analysis and less time on
data wrangling.

This was a challenge when I first started the role, as no data strategy or
governance plan was in place, so it was my responsibility to draft and implement
such plans. The approach I chose for our small team of three was to leverage
existing open-source genomics pipelines (e.g. [`nf-core` community
pipelines](https://nf-co.re/)) where possible, minimising the overhead
maintenance and development associated with in-house pipelines. Costs are
further reduced by using freely available resources such as [MASSIVE
M3](https://docs.massive.org.au/) as our cluster environment and the [ARDC
Nectar Research Cloud](https://ardc.edu.au/services/ardc-nectar-research-cloud/)
as our database host. Across our codebases, I also try to restrict the number of
frameworks and languages used to common competencies in the fields of biology
and bioinformatics (E.g. Bash, R, Python, SQL) to minimise the cost associated
with training.

In the last two years, I have made significant strides in moving our
organisation's data management processes from data awareness to data
proficiency:

1. **Shifted data processing from ad-hoc, local processing with R scripts to a
   consolidated Extract Transform Load (ETL) pipeline** where data from genomic
   pipelines are loaded onto a PostgreSQL database and transformed using DBT
   into cleaned, annotated, and tested datasets.
2. **Improved dataset accessibility** by allowing researchers to
   directly query the CCMA database through
   [`dbplyr`](https://dbplyr.tidyverse.org/) in `R`, without any
   required knowledge of SQL. This minimised the need for researchers to
   download gigabytes of data and process it locally.
3. **Consolidated post-processing scripts that generate datasets into a git
   controlled codebase**, focusing on modular, reusable scripts that follow the
   basics of [UNIX
   philosophy](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html).
4. **Reorganised raw and processed data storage according to a standardised
   schema**, improving accessibility, reducing quota usage (with a reduction of
   `~60 TB` from deduplication), and optimising file formats for tape recall.
5. **Standardised the naming schemes for cell lines** across the organisation to
   improve the searchability of cell lines and ensure that they can act as
   functional natural keys across datasets, while remaining recognisable for
   easy use in lab environments.
6. **Developed several critical Google sheets used by various non-technical teams**
   to track the progress of various assays, keep track of inventories, and
   manage metadata on cell lines. Data validation and protection rules are used
   to ensure data reliability.
7. **Consolidated organisational documents** scattered across local files and
   personal drives into a managed Google shared drive with an ordered file
   structure and tag system that facilities easy file discovery.

I am well versed in bioinformatics, which is a requirement in order
to effectively and accurately process a broad variety of genomic data.
Apart from my love of data engineering, I am also passionate about using
statistics and effective data visualisation to make data-driven
decisions. My other hobbies include:

- 🪓 **Woodworking** (i.e. collecting tools that I may someday use)
- 🕹️ **Gaming** (the more byzantine, the better e.g. Dwarf Fortress, Underrail)
- 📷 **Photography** (I was particularly prolific when I studied agriculture,
and there were plenty of canola fields...)

### Connect

I would love to hear from you! I'm always happy to discuss my experiences and to
hear more about any opportunities.

[![gmail](https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:vikesh.ajith@gmail.com)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/vikeshajith/)

### Languages

![GNU Bash](https://img.shields.io/badge/GNU_BASH-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)
![R](https://img.shields.io/badge/R-276DC3?style=for-the-badge&logo=r&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Nextflow](https://img.shields.io/badge/Nextflow-3ac486?style=for-the-badge)

### Software

[![dbt](https://img.shields.io/badge/dbt-FF694B?style=for-the-badge&logo=dbt&logoColor=white)](https://www.getdbt.com/)
[![Tidyverse](https://img.shields.io/badge/Tidyverse-1A162D?style=for-the-badge&logo=Tidyverse&logoColor=white)](https://www.tidyverse.org/)
[![Slurm](https://img.shields.io/badge/Slurm-1baee9?style=for-the-badge)](https://slurm.schedmd.com/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](#)

[![Visual Studio Code](https://img.shields.io/badge/Visual_Studio_Code-0078D4?style=for-the-badge&logo=visual%20studio%20code&logoColor=white)](#)
[![DBeaver](https://img.shields.io/badge/DBeaver-382923?style=for-the-badge&logo=DBeaver&logoColor=white)](https://dbeaver.io/)

### OS

![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

## Statistics

**Note:** These statistics only apply to commits on public repos. Most
of my recent commits would be to private work-related projects.

[![Top
Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=spikyclip&layout=compact&theme=default)](https://github.com/anuraghazra/github-readme-stats#gh-light-mode-only)
[![Top
Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=spikyclip&layout=compact&theme=ayu-mirage)](https://github.com/anuraghazra/github-readme-stats#gh-dark-mode-only)
