<!--
http://blog.qgis.org/2020/04/26/qgis-grants-5-call-for-grant-proposals-2020/
https://forms.gle/SpyyEStwqtorzB4D9
-->

# Application Form (checklist)

For submission into the 2020 round of QGIS Grants proposals.

- [x] Your name: Sebastian M. Ernst
- [x] Your email: ernst@pleiszenburg.de
- [x] Grant request in Euros: 10k Euro <!-- between 1 Euro and 10,000 Euros -->
- [x] Proposal title: Turning Plugin Management into Actual Package Management
- [x] Proposal details: See [QEP's Abstract](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#abstract-)
- [x] QEP link: https://github.com/qgis/QGIS-Enhancement-Proposals/* / https://github.com/qgist/pluginmanager-qep
- [x] History: <!-- Tell us a little about what work has already been done related to your grant proposal. Are you starting from scratch or are do you plan to build on the work of others? -->
    - I explored options by developing a proof-of-concept QGIS plugin - [see Github repo](https://github.com/qgist/pluginmanager).
    - I tried and studied every bit of proposed technology in terms of figuring out the overall feasibility of the proposed changes.
    - I explored, debugged and documented all relevant sections of QGIS source code (as documented in the QEP).
    - I got in touch with developers through the developer mailing list, exploring design issues and asking questions for gaining a deeper understanding of the subject matter (as documented in the QEP).
- [x] How are you qualified to do this: <!-- Tell us about previous work you have done that will demonstrate that you have the needed skills and enthusiasm to complete this task. -->
    - I have built (i.e. compiled) QGIS from scratch many times and successfully applied changes.
    - I have intensely studied QGIS code base (as documented in the QEP).
    - I develop and maintain QGIS plugins (some open source, some proprietary).
    - I develop and maintain established open source Python packages.
    - 15 years of Python experience, for the past 6 years on a daily & professional basis
    - 7 years of experience in using Qt
    - While I am confident that I can handle the proposed C++ cleanups myself, I would be happy to collaborate with an experienced QGIS C++ developer in this particular matter.
- [x] Implementation schedule / plan: <!-- Lay out for us what will be done when. Please try to tie your work plan to the QGIS release schedule and other key activities in the QGIS project. -->
    - Most of this work will be carried out in July and August of 2020.
    - It is supposed to be finished and tested before the feature freeze of QGIS 3.16 on 2020-09-11.
    - Step 1 (as described in QEP) has mostly been implemented. Open points:
        - Transplant existing work on step 1 into QGIS code base.
        - Finish the unfinished pieces (dependency mechanism cleanup and plugin loading sequence) of step 1. Required time: 2 to 3 days.
        - Implement step 2 (as described in QEP). Required time: Roughly 2 full weeks.
        - Implement tests for new plugin manager module and fix bugs along the way. Required time: Roughly 1 to 2 full weeks.
        - Write documentation. Required time: Roughly 2 to 3 days.
    - Total allocated time for project: 5 weeks full time, including buffers.
    - I am planning to maintain this code long-term.
