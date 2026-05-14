# Resume/CV    										                                                                                                               
Kurt Fehlhauer     										                                                                                                               
email: kfehlhauer@pm.me

Summary
--------
Twenty years building data infrastructure at Fortune 50 scale. Founded the data architecture function at Stellantis's AI division ($60M platform, production agentic AI) and modernized the analytics backbone for Call of Duty at Activision. Hands-on with Rust, Spark, and Databricks; equally comfortable writing production code and presenting to executive leadership.


Experience
--------------
2/2022 - Present
### Chief Data Architect
#### Stellantis: Remote
Founded the enterprise data architecture function at a Fortune 50 automaker. Built the team from zero to 20+ engineers across 3 business units in North America and EMEA; team now requires three engineering managers. Current focus: AI platform architecture and division-wide technical standards with dotted-line oversight of 5 senior engineers.

##### Platform Architecture & Build
- Architected and launched a multi-petabyte data platform on Databricks, Kubernetes, and Delta Lake, processing 500TB/day of vehicle telemetry; platform now serves 500+ analysts, data scientists, and engineers
- Secured $60M annual budget through Stellantis corporate governance to fund the buildout of connected services data infrastructure 
- Scaled the function to require three engineering managers — one for DevOps/platform and two for data engineering — across hiring, mentorship, and organizational design
- Built a Rust-based time-series data quality monitoring system on DataFusion and delta-rs that runs on a single Docker container, replacing a Spark/Databricks cluster-based implementation; cut monitoring infrastructure costs 80%+ by eliminating cluster overhead while maintaining throughput on multi-petabyte Delta Lake tables 

##### AI Platform & Architecture Leadership
- Designed and shipped a production agentic AI system that autonomously maps the architecture of 1,000+ data and analytics applications using a hybrid local/cloud LLM pipeline (Rig framework, Claude Opus, Qwen models); compressed a 3-month manual discovery process to 7 hours, saving an estimated 10 engineer-months and $250K in modernization planning 
- Set divisional data architecture direction across three engineering teams through dotted-line oversight, technical review authority, and platform standards ownership 

##### Acting Head of Data Governance (March 2025 – January 2026) 
- Stepped into divisional leadership role during executive transition, managing 5 direct reports and 8 total staff across data discovery, AI enablement, and democratization initiatives while maintaining architecture responsibilities 
- Authored the Stellantis enterprise Data and AI tenets, adopted company-wide 
- Established enterprise data governance framework and PII protocols spanning GDPR, CCPA, and other regional regulations, unblocking AI initiatives previously stalled by compliance review 
- Built data discovery and cataloging system over untyped vehicle signal data using Rust, DataFusion, and DuckDB
_____


5/2019 – 2/2022	
### Senior Manager, ETL
#### Activision Publishing: Remote
Managed a data engineering team supporting the Call of Duty franchise and studio analytics, driving platform modernization initiatives that improved data processing times by 50% while ensuring regulatory compliance.
-	Consolidated a fragmented big data ecosystem (Qubole, Hive, Presto, Redshift) into a unified Databricks platform, eliminating vendor sprawl and improving vendor management by 40%
-	Implemented Astronomer Airflow deployment, standardizing workflow orchestration and reducing deployment complexity
-	Engineered a high-performance third-party data ingestion framework using functional programming paradigms (Cats/ZIO, Circe, http4s) on Kubernetes
-	Mentored 20+ engineers in modern data stack technologies (Airflow, Kubernetes, Scala, Spark).
_____


6/2014 – 4/2019
### Lead Database Architect
#### Activision Publishing: Boulder, CO/Remote
Promoted from Lead Database Architect to Senior Manager across 7+ years modernizing the data infrastructure powering Call of Duty and studio analytics. Drove the migration from fragmented legacy systems (Qubole, Hive, Presto, Redshift) to a unified Databricks/Delta Lake platform; championed company-wide adoption of Spark and Airflow.

##### Senior Manager, ETL – Demonware (May 2019 – February 2022)
- Managed a team of 5 data engineers supporting Call of Duty franchise analytics and studio operations across 5 global studios
- Consolidated a fragmented stack (Qubole, Hive, Presto, Redshift) onto a unified Databricks platform, cutting platform team operational overhead by 80 hours/week
- Standardized workflow orchestration on Astronomer Airflow, reducing average pipeline deployment time from 1 day to 3 hours across 35 production DAGs
- Engineered a high-throughput third-party data ingestion framework in Scala using Cats/ZIO, Circe, and http4s on Kubernetes, processing millions of events/day from multiple game titles

##### Lead Database Architect – Analytical Services (June 2014 – April 2019)
- Architected the big data platform supporting Call of Duty's hundreds of millions of registered users, processing petabytes of player telemetry and game performance data
- Drove company-wide adoption of Apache Spark and Airflow, positioning Activision among the early enterprise adopters; reduced batch processing windows by more than 50% over MapReduce jobs
- Initiated the Databricks/Delta Lake migration that completed under the successor role, retiring multiple legacy systems and reducing infrastructure spend by $2.3M annually
_____


1/2013 – 6/2014
### Senior Consultant
#### FICO: Remote
Delivered credit and retail applications for financial institutions and pharmaceutical companies.
-	Rescued and delivered a voice/SMS gateway system for the nation's largest bank after inheriting a yearlong stalled project, preserving $10M+ annual client relationship and preventing potential contract termination of other services
-	Designed and implemented recommendation engines for major pharmaceutical companies, optimizing drug treatment awareness by 15%
_____


10/2011 – 12/2012
### Senior ETL Architect
#### Productive Data Solutions: Denver, CO
Provided strategic guidance and expert knowledge that implemented ETL solutions for multiple states to power their healthcare exchanges.

- Designed and implemented a hybrid ETL architecture combining Pentaho Data Integration with Python automation, processing millions of records daily and reducing data processing errors by 40%
- Developed a HIPAA-compliant healthcare reporting system using Python, ensuring zero privacy violations while enabling real-time analytics for patient records
- Established automated testing framework using Linux shell scripting, mentoring QA teams on DevOps practices that reduced manual testing efforts by 70%
- Led technical training initiatives for development teams on advanced Python ETL techniques, improving code quality and standardizing best practices across projects
_____

3/2006 – 10/2011
### Software Architect
#### Transzap: Denver, CO
Drove enterprise software architecture transformation at high-growth oil and gas fintech startup, delivering scalable e-payables solutions that contributed to the company's recognition in Deloitte Fast 500 and supported millions in transaction processing volume.

- Introduced cutting-edge columnar database technology (Vertica) to the enterprise stack, offloading analytical workloads from transactional systems and improving query performance by 90%
- Led SSAS cube modernization initiative, migrating complex MDX-based analytics to SQL-accessible Vertica platform, democratizing data access for business analysts
- Built a comprehensive ETL infrastructure using Pentaho Data Integration, processing financial transactions daily while maintaining 99.9% data accuracy
- Engineered SQL Server Integration Services pipelines for enterprise data warehouse, enabling real-time business intelligence for executive decision-making
- Optimized mission-critical C# application (Spendworks) startup performance, achieving a 95% reduction in load times (from minutes to seconds), dramatically improving user adoption rates
- Mentored development team on emerging technologies and architectural best practices, establishing technical standards that supported company scaling from startup to enterprise
_____


7/2000 – 3/2006
### Application Architect
#### Calpine: Fort Collins, CO
Led enterprise architecture at Fortune 500 energy company, pioneering real-time power plant analytics and telemetry systems that optimized operations across a fleet of natural gas facilities while driving company recognition as InformationWeek Top 100 Innovator.

- Pioneered early adoption of Microsoft .NET technologies at enterprise scale, collaborating directly with Microsoft development teams on C# language evolution and establishing Calpine as an industry technology leader
- Managed project budgets up to $600K and established PMO standards for timeline management, delivering 95% of projects on time and under budget across the IT portfolio
- Architected a comprehensive business intelligence platform using ASP.NET, SQL Server, and SSAS, enabling real-time analysis of natural gas and electric power sales data
- Developed custom data mapping platform using C#, ADO.NET, Oracle, and OSI PI, standardizing data integration across heterogeneous industrial control systems
- Enhanced legacy C++/MFC libraries to support dynamic contract period management, improving operational flexibility for diverse power purchase agreements
_____


12/1999 – 7/2000
### Systems Analyst II
#### City of Thornton: Thornton, CO
Led municipal technology modernization initiatives for a growing suburban city, implementing mission-critical systems serving 77K+ residents while establishing software development best practices and mentoring technical staff on emerging enterprise technologies.

- Provided technical leadership and mentorship to MIS staff on advanced programming technologies, including C++, COM, MTS, and ASP, elevating team capabilities in enterprise application development
- Modernized legacy municipal systems serving police, fire, utilities, and administrative departments, ensuring 99.9% uptime for citizen-facing services and emergency response systems
- Collaborated with department heads to align technology solutions with operational requirements, ensuring seamless integration across police dispatch, utilities management, and citizen services platforms
- Created technical documentation, creating a sustainable IT operations framework for the City of Thornton
_____


1/1999 – 12/1999
### Information Technology Lead
#### VantagePoint Network: Fort Collins, CO
Led software engineering that pioneered the development of a groundbreaking precision agriculture platform, creating one of the industry's first web-based agricultural technology solutions that enabled crop professionals to optimize yields while reducing environmental impact across thousands of farming operations.

- Designed a comprehensive soil analysis and management platform integrating C++ ATL COM objects with enterprise Oracle databases, providing farmers with data-driven insights for optimized fertilizer application and soil health monitoring
- Created a web-based crop record management system using ASP and Oracle, digitizing paper-based processes and reducing administrative overhead by 60% for agricultural operations
- Led quality assurance standardization initiative, collaborating with QA teams to establish bug tracking, testing protocols, and code quality standards that reduced production defects by 40%
- Partnered with database architects to design a scalable agricultural data warehouse supporting complex crop rotation analysis, field mapping, and yield prediction algorithms
_____


2/1995 – 1/1999
### Programmer/Analyst
#### State Farm Insurance Companies: Bloomington, IL
Delivered enterprise-scale insurance technology solutions for a Fortune 50 company, architecting mission-critical systems that processed millions of policies annually while mentoring development teams and establishing coding standards across business units.

- Designed high-performance data replication system using C++, DB2, and MQ Series, synchronizing marketing data across 5,000+ agent locations nationwide with 99.9% reliability
- Led after-hours technical training initiatives in C++ programming, developing internal expertise
- Provided critical vendor support in debugging a complex MFC C++ life insurance illustration application, ensuring accurate actuarial calculations for a multi-billion dollar life insurance portfolio
- Enhanced AionDS-based expert system for auto policy pricing, implementing advanced business rules that improved pricing accuracy by 25% and reduced underwriting exceptions
_____


1/1996 - 12/1996
### C++ Instructor
#### Heartland Community College: Bloomington, IL
Delivered comprehensive object-oriented programming education to 50+ students while maintaining a full-time State Farm position, developing curriculum and teaching methodologies that achieved a 95% student success rate in C++ programming concepts.

- Designed and delivered a comprehensive C++ curriculum covering advanced object-oriented programming principles, data structures, and software engineering best practices for computer science and engineering students
- Created teaching materials and practical assignments that bridged academic concepts with industry applications, drawing from my development experience
- Applied State Farm enterprise development experience to provide students with practical insights into the commercial software development lifecycle
_____


5/1993 – 1/1995
### Computer Operator
#### Rockwell Automation Allen – Bradley: Mequon, WI
Operated mission-critical mainframe systems for Fortune 500 industrial automation leader, contributing to successful ISO 9000 certification initiatives.

- Operated and maintained enterprise mainframe systems processing critical manufacturing data for Rockwell's worldwide industrial automation operations, supporting $2B+ annual revenue
- Led automation initiative for mainframe application scheduling, designing workflows that increased operational efficiency and reduced after-hours support requirements
- Supported multiple successful ISO 9000 quality certification audits through meticulous documentation of mainframe operational procedures
- Collaborated with systems analysts and programmers on troubleshooting complex system issues
- Built foundational expertise in enterprise system operations and quality management that supported transition into a software engineering career

			
Education
-----------
University of Wisconsin - Milwaukee  
Bachelor of Business Administration - 1994  
Major: Management Information Systems  
