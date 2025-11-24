---
title: 'DBCLS BioHackathon 2025 report: MCP server tools with RDF shapes'
title_short: 'BioHackJP25: MCP servers with shapes'
tags:
  - Semantic web
  - Ontologies
  - Shapes
  - ShEx
  - LLM
  - Model Context Protocol
authors:
  - name: Jose Emilio Labra Gayo
    orcid: 0000-0001-8907-5348
    affiliation: 1
  - name: Yasunori Yamamoto
    orcid: 0000-0002-6943-6887
    affiliation: 2
  - name: Akira R. Kinjo
    orcid: 0000-0002-4006-8208
    affiliation: 5
  - name: Andra Waagmeester
    orcid: 0000-0001-9773-4008
    affiliation: 3
  - name: Javier Millán Acosta
    orcid: 0000-0002-4166-7093
    affiliation: 4
  - name: Shuichi Kawashima
    orcid: 0000-0001-7883-3756
    affiliation: 2
  - name: Yoko Okabeppu
    affiliation: 6
  - name: Julia Koblitz
    orcid: 0000-0002-7260-2129
    affilation: 3
  - name: Samuel Bustamante-Larriet
    orcid: 0009-0005-8631-2682
    affiliation: 1    
  - name: Daniel Fernández-Álvarez
    orcid: 0000-0002-8666-7660
    affiliation: 1
affiliations:
  - name: University of Oviedo, Spain
    index: 1
  - name: Database Center for Life Science, Joint Support-Center for Data Science Research, Research Organization of Information and Systems
    index: 2
  - name: Amsterdam University Medical Center
    index: 3
  - name: Maastricht University
    index: 4
  - name: Anima Machina G.K.
    index: 5
  - name: OKBP inc.
    index: 6
date: 19 September 2025
cito-bibliography: paper.bib
event: BH25JP
biohackathon_name: "DBCLS BioHackathon 2025"
biohackathon_url:   "https://2025.biohackathon.org/"
biohackathon_location: "Mie, Japan, 2025"
group: BH25-MCP-server-shapes
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-japan/BH25-MCP-server-shapes
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Jose Labra \emph{et al.}
---

# Introduction

The Model Context Protocol (MCP) [@citesForInformation:mcp_anthropic] establishes an interface between Large Language Models (LLMs) and external utilities. 
MCP servers allow developers to expose specialized tools through a uniform schema enhancing interoperability accross different systems transforming the integration from a many-to-many dependency graph to a uniform interface.
It can increase the usability of LLMs which can interact directly with structured knowledge bases,
reducing the overhead required to bridge probabilistic reasoning with deterministic tools.
One possible application of MCP servers is to offer access to RDF data portals which expose SPARQL endpoints,
offering a natural language interface which is complemented with a deterministic SPARQL-based query engine. 
While RDF and SPARQL provide a highly expressive knowledge representation, the flexibility of their schema-agnostic approach can result in structural opacity, 
hindering reliable automated consumption. Shape Expressions (ShEx) have been proposed as a formal schema language that can describe the required graph topology of an RDF data portal. 

In this paper, we present a report of the project that we developed as part of our participation in the [DBCLS BioHackathon 2025](https://2025.biohackathon.org/), which was focused on the exploration of using MCP servers with RDF data portals and Shape Expressions.
During the project, we also identified other challenges like the comparison between different Shape Expressions that can be extracted from existing SPARQL endpoints or the analysis of the evolution of ontologies using their shapes.



## RDF Portal MCP server (TogoMCP)

The RDF Portal MCP server ("TogoMCP" for short) connects an LLM (we exclusively used Claude Sonnet 4 in BH25) to the RDF Portal SPARQL endpoints. Given natural language queries, the LLM translates them into SPARQL queries. TogoMCP sends those SPARQL queries to the RDF Portal and the obtained query results are sent back to the LLM for further interactions.
Before BH25, TogoMCP has already incorporated several databases provided at the RDF Portal, such as UniProt, PDB, ChEMBL, PubChem, to name a few. 

## The MIE file

To make SPARQL generation more accurate and effective, TogoMCP utilizes a special file, called a MIE file, for each database. The MIE file for a database contains the metadata, a biologically relevant subset of ShEx and VoID schemas, as well as examples of RDF subgraphs and SPARQL queries, all concisely annotated in natural language (English). When a user provides a prompt that requires SPARQL queries, the LLM will (usually) calls an MCP tool named **describe_rdf_schema** with the database name as the argument, which presents the MIE file to the LLM. The MIE files are semi-automatically generated using the LLM and a hand-crafted template MIE file. An MCP prompt with a database name argument is also a part of TogoMCP. When BH25 started, there was no specification for the MIE file. The structure of the MIE template was ad hoc, and we didn't know which parts of the MIE file was critical. Thus, the objectives of working team included finding out the features essential for accurate and effective SPARQL generations and explicitly specifying them in the template file.

# Day 1

## TogoMCP
We added Glycosmos, along with its MIE file, to TogoMCP. Unlike other databases, the Glycosmos interface uses its original SPARQL endpoint (https://ts.glycosmos.org/sparql) rather than the one provided by the RDF Portal so it has access to the latest version of Glycosmos. The process of creating the Glycosmos MIE file is provided at https://claude.ai/share/6ed633b7-24c6-4843-98c9-eb4506c5c156. The resulting MIE file is uploaded to the GitHub: https://github.com/arkinjo/RDFPortal-MCP/blob/BH25/mie/glycosmos.yaml. We tested Glycosmos with a few biological questions:

- Glycan Biosynthesis Pathway Evolution: https://claude.ai/share/3dfcd4d2-205a-493d-9a6a-e9edf3df6860
- Glycosylation-Related Genes in Cancer: https://claude.ai/share/4a9b2cdc-edc4-429c-9b8e-29681e747247

Members from another working group (Glycosmos MCP server) confirmed that the results were reasonable.

# Day 2
## TogoMCP/MIE file
One of the crucial ingredients of the MIE file is the ShEx-like schema, which contains a biologically relevant subset of the ShEx (shape expressions). Currently, this subset is more or less randomly determined by the LLM during its exploration of the RDF graphs. As a results, it is highly possible that important classes and properties are missing in the resulting MIE file, which in turn deteriorates the quality of LLM-generated answers to biological queries. To avoid this problem, we considered the use of RDF-Config. RDF-Config can generate a SheX schema from a set of hand-crafted RDF-config (YAML) files. As such, we can explicitly specify a biologically relevant subset of the ShEx schema. We may use this feature of RDF-Config to craft a better prompt for creating the MIE file. However, making the RDF-config files can be a tedious task. Thus, we tried to use an LLM to help this process by creating a prompt for the LLM that contains a template of the RDF-Config YAML files. 

- The RDF-Config template (tentative): https://github.com/arkinjo/RDFPortal-MCP/blob/BH25/rdf-config/template.yaml
- Generating the Glycosmos RDF-Config using Claude: https://claude.ai/share/2f903a1c-eb12-479e-ad22-96b9ca13be90

The resulting RDF-confing file looks promising. However, we found that the LLM sometimes gave outputs that deviated from the specification of the RDF-Config file. Further experimentation is necessary to produce strictly well-formed RDF-Config files.


# Day 3
## TogoMCP/MIE file
Upon the request from the Microbial Knowledge Bases group, we integrated BacDive and MediaDive into TogoMCP. As they had manually curated RDF-Config files, we could also have used the biologically relevant subsets of the ShEx schemas of these databases, but this was not done at this time (it was eventually done on Day 5). In addition, they prepared a list of biological questions:
### BacDive Questions
- Show me the type strain of Belnapia moabensis.
- Which strains belong to the order Rhizobiales?
- Does strain DSM 16746 form spores?
- What is the cell shape and Gram stain of Escherichia coli?
- Which strains are motile and aerobic?
- List bacteria that produce orange pigments.
- What temperature and pH ranges support growth of Saccharolobus solfataricus?
- Which strains grow at 30 °C but not at 15 °C?
- What is the GC content of Corynebacterium glutamicum?
- Give me the 16S rRNA accession number for DSM 16746.
- Which strains have genome assemblies in PATRIC?
- What is the biosafety level of DSM 639?
- Which strains are pathogenic to humans?
- Are there strains that are susceptible to penicillin but resistant to ampicillin?
- List strains that can degrade toluene.
- Which strains from the family Flavobacteriaceae are moving by gliding and have alpha hemolytic activity?
- Which strains isolated from soil produce endospores?
- Show me the strains from a marine environment and their mean optimal temperature and pH.
- Show me all strains isolated from the host "Zea mays" and their pathogeneticity against plants.
- Regarding enzyme activity, which strains have alcohol dehydrogenase activity and grow at high temperature?
- Which strains can utilize both, glucose and nitrate?We used these questions to create the MIE files. That is, we instructed the LLM to explore the RDF graph structure deeply enough to be able to answer these questions. 

### MediaDive Questions
- What is the composition of DSMZ Medium 119?
- Which ingredients are used in MRS Medium?
- Does Medium 119 contain KH2PO4?
- Which equipment is required for preparing Medium 119?
- Give me the CAS number and synonyms for KH2PO4.
- Which medium ingredients are frequently optional?
- What is the pH and temperature for cultivating strains from Lactobacillus in MRS Medium?
### Combined BacDive and MediaDive Questions
- Which strains are known to produce lactic acid and grow in MRS Medium?
- List strains that can be cultivated in Medium 1 and are motile.
- Which pathogenic strains can be grown using glucose as a carbon source?

We can include these questions in the prompts to create the MIE files so that the LLM explore the RDF graphs as deeply as necessary to answer these questions. We then tried a few queries: 
- https://claude.ai/share/17692599-c73a-49ad-b53f-cbdcf3aee3a6
- https://claude.ai/share/94400107-b8bb-49fa-adfa-dff7bbc2d947

The Microbial KB group examined these results and confirmed that they were reasonable, although not perfect.

# Day 4 
## TogoMCP/MIE file
### The MIE file specification
We discussed on the specification of the MIE file and agreed that the main contents should include the following items:
- schema_info (Metadata on the RDF graphs)
- prefixes (prefixes used in the graphs; This might be unnecessary once the shape_expressions section is replaced with pure ShEx)
- shape_expressions (This may be replaced with the plain vanilla ShEx format?)
- RDF examples (concrete examples of common RDF graph patterns)
- SPARQL examples (This section should include natural language questions and corresponding SPARQL queries)
- cross-reference SPARQL examples (cross-references are particularly important for connecting databases, so we prepare an independent section.)
- data_statistics (statistics useful for optimizing SPARQL queries)
### Feeding the raw ShEx into the LLM
BacDive and MediaDive come with the ShEx files generated from manually curated RDF-Config files. We attempted to feed the ShEx files directly into the LLM to see the significance of this information. Here are the results of example queries with or without ShEx:
- With ShEx: https://claude.ai/share/9f270856-ab09-4147-a970-e176742f1e06
- Without ShEx: https://claude.ai/share/a86cfbfc-cc38-4e3d-8746-74306ebea632 

We confirmed that the use of ShEx significantly improved the results.

# Day 5
## TogoMCP/MIE file

We updated all the MIE files for TogoMCP to confirm the specification decided on the previous day. In particular, the MIE files for BacDive and MediaDive, the RDF-Config-generated ShEx and biological questions were also given to the LLM so that the resulting MIE files incorporated all the ShEx components and the LLM can answer the given questions. We found that creating the MIE files with ShEx and biological questions can take much longer time and some additional prompts, and the resulting MIE files had larger sizes. For example, BacDive's MIE file now has more than 2,000 lines, which may be a little too large than intended. This is partly because the MIE file contained all the given biological questions and corresponding SPARQL queries, in addition to the other SPARQL query examples required by the specification. We then tested the LLM with a few questions. Here are a few examples:

- Composition of a medium: https://claude.ai/share/45982d86-1448-4e8d-8983-7d18610f00e7
- CO2 containing media with inorganic compounds: https://claude.ai/share/f57ef84e-8d8d-4074-8acd-18cda078b224

For the latter question, the Microbial KG group found two media that were previously unknown as containing CO2 and free of organic compounds.

# Accomplishments and challenges identified

- Extended TogoMCP to include Glycosmos, BacDive, and MediaDive.
- We have discovered the importance of biologically relevant questions for better in-context learning of the LLM.
- We have confirmed the power of biologically relevant subset of ShEx schemas for high-quality SPARQL generation.

# Conclusions and future work

By leveraging ShEx as a machine-readable contract, an MCP server can translate natural language requests from the LLM into syntactically and semantically validated SPARQL queries, transforming a black-box data source into an introspectable, usable tool for agentic systems.
