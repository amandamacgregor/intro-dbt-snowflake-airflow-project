Intro to some DE tooling.
Referenced https://www.youtube.com/watch?v=OLXkGB7krGo
Need to figure out why my models didn't materialize as tables, only views, as expected from the config file - I had to expressly tell them per model.

Used Snowflake data, set up a user and role and db and wh and schema.
Installed and used dbt core w/ vs code to create views, and transform data with models, and add some generalized tests and singular tests.
Ran, populated the views and tables within Snowflake.

Installed Astro, used it to spin up Airflow.
Connected it to dbt project and Snowflake account.
Tasks ran successfully when I triggered the dag - after a lot of troublshooting w/ different UI than the video showed re: snowflake connection.

Not complicated, focused on tooling here vs. what we did in the SQL scripts and tranformations.
Not sure why we'd use Airflow on this, or if we'd need to just pull changes to the data, whatever - not focused on use case, just getting some familiarity.
