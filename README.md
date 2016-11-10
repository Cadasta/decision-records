# Cadasta Development Decision Records

This repository is a collection of records for significant decisions that the development team made regarding the development of the Cadasta platform. The decisions may affect the architecture, dependencies, development techniques and the design of the platform. The concept and structure of decision records are borrowed form [Architecture Decision Records](http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions).

We will create one document per decision. File names follow the pattern `DR_NNNN_short-title.md`, i.e. the decision record for asynchronous file processing could be `DR_0001_asyc-file-processing.md`. Decision records are written in Markdown. 

Decision records will be numbered sequentially; numbers will not be reused. If an existing decision is reversed, replaced by a new decision or becomes deprecated a new decision record must be created, and a linked reference to the related record is added to both documents. The status of the existing document must change accordingly. We will never deleted or change a decision record once it has been accepted. 

## Structure of a decision record

All decision records should be created using the following structure:

- **Title**: A short descriptive title; e.g. "0001: Architecture for asynchronous processing of files."
- **Context**: This section describes the current situation that leads to the decision; for instance, partner requirements or current technological shortcomings. 
- **Decision**: Here we describe how we respond to the context, the choices we made and why we made those choices. 
- **Consequences**: Describe how the decision affects the current context. All consequences should be listed here, both positive and negative. 
- **Status**: This is the status of the document; it is one of `draft proposal`, `accepted` or `deprecated`. If another decision replaces this decision, place a linked reference to the replacing decision record next to the status. 

## Proposing a new decision

1. Create a new branch off master. Give the branch the same name that the document will have `dr_0001_asyc-file-processing`. Before creating a new branch, check if there are any existing draft branches and select the record’s sequential number accordingly. 
2. In the new branch, create a new document using the branch name as the document’s file name and write up your decision proposal. 
3. Once your proposal is ready for review, open a pull request. 
4. Everyone in the dev team can then review your proposal and ask for clarification or changes. 
5. The proposal can be accepted through consent of the dev team in a dev team meeting. Then the PR will be merged, and the status of the document is changed to `accepted`. 
