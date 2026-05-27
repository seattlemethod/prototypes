## Read Me

These CSV files contain demo data from an accounting system.  There are 22,288 general ledger entries or about 3,000 transactions that are "mapped" to standard report line items and standard business events represented using an XBRL taxonomy.  This is the XBRL taxonomy:

[MINI 2026](https://xbrlsite.azurewebsites.net/2026/reporting-framework/mini/base-taxonomy/mini_ModelStructure.html)

The objective is for a software application to take the general ledger transactions and, using the XBRL taxonomy above, to generate an XBRL report including a balance sheet, income statement, cash flow statement, statement of changes in equity, a roll forward of each balance sheet line item, a trial balance, and a summary of business event information.

Here is an example of that report that would be expected to be generated:

[MINI 2026 Reference Implementation #2](https://xbrlsite.azurewebsites.net/2026/reporting-framework/mini/ref-num/index2.html)

There are a few issues that need to be dealt with:

1. **Opening Balance**: The transactions contain opening balances that were in the demo accounting system data.  The "mini:OpeningBalance" is not included in the MINI 2026 taxonomy.  Not exactly sure how to address opening balances for the long term.
2. **Policies and Other Text Disclosures**: My example includes supplemental policies and disclosures such as the basis of reporting, nature of business, and revenue recognition policy. Obviously, that information is not included in the transactions data.  Policies and other such text based disclosures that do not come from the accounting system itself should be appended to the report using a separate process.
3. **Subclassifications**: This example was created before the notion of subclassifications was addressed.  Subclassifications can be dealt with by using the lowest level line item as the line item code and then let software figure out the "branches" above.  For now, I would say simply leave the subclassifications disclosures out; they can be manually added via a separate process.
4. **Traceability**: What would be REALLY GREAT would be to show the [traceability](https://seattlemethod.blogspot.com/2025/12/traceability.html) that is possible if all the semantic fragmentation is eliminated from accounting transactions.
5. **Provenance**: Another excelent thing to show would be [provenance](https://seattlemethod.blogspot.com/2026/02/provenance.html) related features.  Some, but not all, provenance information is in the CSV file of general ledger transactions.
6. **Subsidiary Ledgers**: This demonstration is obviously for the general ledger only.  All subsidiary ledgers would work in a similar manner; they are simply more details.  Showing ONE subsidiary ledger would be more work, but it would answer the obvious question in a currious accountant's mind, "What about subsidiary ledger information?"
7. **Interconnections**: The Auditchain Luca Suite does not show the capability to navigate around in a report leveraging the incerconnections of information in the report.  Pesseract does show that.  Having that capability would be a very good thing.
8. **LLM Enabled Audit**: The general ledger transactions have a lot of interesting possibilities for examining transaction posting patterns and such. Showing that within the context of a financial report would be most excellent.
9. **Verification**: Verifying the report model and report per the MINI 2026 base reporting framework would be excellent.
10. **Framework**: This should NOT BE A ONE OFF POINT SOLUTION! This should be a framework; a framework which could process a report created using [any of these financial reporting frameworks](https://seattlemethod.blogspot.com/2026/01/reference-reporting-frameworks.html).  Why? What is the benefit of US GAAP and IFRS requiring different software applications to work just because they are different reporting frameworks?  There should be ONE MECHANISM that works for any financial reporting framework.

Showing all this in working software would help accountants understand the possibilities here. This is not even addressing the possibility of leveraging [Data Centric Accounting](https://xbrlsite.azurewebsites.net/seattlemethod/dca/dca_ModelStructure.html) to help bookkeepers and accountants in the posting of transactions (i.e. rather than the standard line item information and the standard business event information being provided as part of the demonstration).  These can be separate processes; but obviously it would be significantly better and even more compelling if they were combined into one process.
