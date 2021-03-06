Developer Guide
===============

Database Design
---------------

Knitter database keeps basic details of a user in a social media page. PostgreSQL and Psycopg2 is used for design of
this database. We have an PostgreSQL server on Bluemix for the application. This database is built on user details,
sales details, item details, knot details, group and event details, book details and poll details. These tables
generate the main structure of Knitter. Knitter gives user lots of opportunities with its wide database. The entities
of this database keep necessary variables for a user as it almost gathers the options that Twitter, Facebook,
Amazon & Goodreads offer.

User and Knot tables are essential tables for the database design. The other tables mainly references to these tables.

.. image:: /images/er.jpg
    :alt: er diagram
    :width: 720px
    :height: 630px
    :align: center

.. centered:: *ER Diagram of Knitter*


Code
----

In database.py file we have a function which creates all the tables. We call this function in /initdb directory. This
directory can be reached from login page by pressing Initialize DB button. It initializes the tables and return to login
page.

.. code-block:: python

    def database_initialization():
        database.create_tables()
        return redirect(url_for('site.login_page'))



.. note:: We did not create a method to drop tables. We just use drop database command from pgAdmin if we need to.

The tables are created in *create_tables()* function of database class. In this function all tables are created in order
we first created tables which do not reference to any other table such as User table. These tables constitute the main
structure. Creation of tables can be seen below.

.. code-block:: python

    def create_tables(self):
        with dbapi2.connect(self.config) as connection:
            cursor = connection.cursor()

            # ----------- Can Altinigne - USERS TABLE ----------------------

            query = """CREATE TABLE IF NOT EXISTS USERS (
                                      USER_ID SERIAL PRIMARY KEY,
                                      USERNAME varchar(20) UNIQUE NOT NULL,
                                      USER_PASSWORD varchar(255) NOT NULL,
                                      PROFILE_PIC varchar(255) NOT NULL,
                                      COVER_PIC varchar(255) NOT NULL,
                                      MAIL_ADDRESS varchar(50) NOT NULL,
                                      REGISTER_DATE date NOT NULL
                                    )"""

            cursor.execute(query)

            # ----------- Can Altinigne - CITIES TABLE ----------------------

            query = """CREATE TABLE IF NOT EXISTS CITIES (
                                                CITY_ID SERIAL PRIMARY KEY,
                                                CITY_NAME varchar(50) NOT NULL,
                                                DISTANCE_TO_CENTER integer NOT NULL,
                                                COUNTRY varchar(3) NOT NULL
                                                          )"""

            cursor.execute(query)

            # ----------- Can Altinigne - CURRENCY TABLE ----------------------

            query = """CREATE TABLE IF NOT EXISTS CURRENCIES (
                                              CURRENCY_NAME varchar(3)
                                              PRIMARY KEY UNIQUE NOT NULL,
                                              CURRENCY_TO_TL numeric(10,2) NOT NULL,
                                              LAST_UPDATE date
                                           )"""

            cursor.execute(query)

            # ----------- Can Altinigne - ITEMS TABLE ----------------------

            query = """CREATE TABLE IF NOT EXISTS ITEMS (
                                     ITEM_ID SERIAL PRIMARY KEY UNIQUE NOT NULL,
                                     ITEM_NAME varchar(50) NOT NULL,
                                     ITEM_PICTURE varchar(255) NOT NULL,
                                     ITEM_PRICE numeric(10,2) NOT NULL,
                                     ITEM_DESCRIPTION text,
                                     ITEM_CURRENCY varchar(3)
                                     REFERENCES CURRENCIES(CURRENCY_NAME)
                                               )"""

            cursor.execute(query)

            # ----------- Can Altinigne - SALES TABLE ----------------------

            query = """CREATE TABLE IF NOT EXISTS SALES (
                                         SALE_ID SERIAL PRIMARY KEY,
                                         SELLER_ID INTEGER REFERENCES USERS(USER_ID)
                                         ON DELETE CASCADE,
                                         ITEM_ID INTEGER REFERENCES ITEMS(ITEM_ID)
                                         ON DELETE CASCADE,
                                         CITY_ID INTEGER REFERENCES CITIES(CITY_ID)
                                         ON DELETE CASCADE
                                         ON UPDATE CASCADE,
                                         START_DATE date NOT NULL,
                                         END_DATE date NOT NULL
                                       )"""

            cursor.execute(query)

            # ----------- Can Altinigne - USER_DETAIL TABLE ----------------------

            query = """CREATE TABLE IF NOT EXISTS USER_DETAIL (
                                         USERNAME varchar(20) REFERENCES USERS(USERNAME)
                                         ON DELETE CASCADE
                                         ON UPDATE CASCADE,
                                         U_NAME varchar(30) NOT NULL,
                                         U_SURNAME varchar(30) NOT NULL,
                                         CITY_ID INTEGER REFERENCES CITIES(CITY_ID)
                                       )"""

            cursor.execute(query)

            # ----------- Tolga Bilbey - KNOTS TABLE ----------------------

            query = """CREATE TABLE IF NOT EXISTS KNOTS(
                        KNOT_ID SERIAL PRIMARY KEY,
                        OWNER_ID INTEGER NOT NULL REFERENCES USERS(USER_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE,
                        KNOT_CONTENT TEXT,
                        LIKE_COUNTER INTEGER DEFAULT 0,
                        REKNOT_COUNTER INTEGER DEFAULT 0,
                        IS_GROUP BOOLEAN NOT NULL,
                        POST_DATE DATE NOT NULL
                    )"""

            cursor.execute(query)

            # ----------- Tolga Bilbey - EVENTS TABLE ----------------------

            query = """CREATE TABLE IF NOT EXISTS EVENTS(
                            EVENT_ID SERIAL PRIMARY KEY,
                            OWNER_ID INTEGER NOT NULL,
                            EVENT_CONTENT TEXT,
                            EVENT_START_DATE DATE NOT NULL,
                            EVENT_END_DATE DATE NOT NULL,
                            IS_USER BOOLEAN NOT NULL
                    )"""

            cursor.execute(query)

            # ----------- Tolga Bilbey - EVENT-PARTICIPANTS TABLE ----------------------

            query = """CREATE TABLE IF NOT EXISTS EVENT_PARTICIPANTS(
                            EVENT_ID INTEGER NOT NULL REFERENCES EVENTS(EVENT_ID)
                            ON DELETE CASCADE
                            ON UPDATE CASCADE,
                            PARTICIPANT_ID INTEGER NOT NULL REFERENCES USERS(USER_ID)
                            ON DELETE CASCADE
                            ON UPDATE CASCADE
            )"""

            cursor.execute(query)

            # ----------- Ozan ATA - LIKE-REKNOT ----------------------

            query = """CREATE TABLE IF NOT EXISTS LIKE_REKNOT(
                        KNOT_ID INTEGER references KNOTS(KNOT_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE,
                        USER_ID INTEGER references USERS(USER_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE,
                        IS_LIKE BOOLEAN
                    )"""

            cursor.execute(query)

            # ----------- Ozan ATA - POLLs ----------------------

            query = """CREATE TABLE IF NOT EXISTS POLLS(
                POLL_ID SERIAL PRIMARY KEY,
                OWNER_ID INTEGER references USERS(USER_ID)
                ON DELETE CASCADE ON UPDATE CASCADE,
                POLL_CONTENT varchar(255) NOT NULL,
                POLL_OPTION_1_CONTENT varchar(255) NOT NULL,
                POLL_OPTION_1_COUNTER INTEGER DEFAULT 0,
                POLL_OPTION_2_CONTENT varchar(255) NOT NULL,
                POLL_OPTION_2_COUNTER INTEGER DEFAULT 0,
                START_DATE date NOT NULL,
                END_DATE date NOT NULL
                )"""

            cursor.execute(query)

            # ----------- Ozan ATA - USER-POLL ----------------------

            query = """CREATE TABLE IF NOT EXISTS USER_POLL(
                POLL_ID INTEGER references POLLS(POLL_ID)
                ON DELETE CASCADE ON UPDATE CASCADE,
                USER_ID INTEGER references USERS(USER_ID)
                ON DELETE CASCADE ON UPDATE CASCADE
                )"""

            cursor.execute(query)

            # ----------- ilknur Meray - USER_INTERACTION TABLE ------------------

            query = """CREATE TABLE IF NOT EXISTS USER_INTERACTION(
                        BASE_USER_ID INTEGER REFERENCES USERS(USER_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE,
                        TARGET_USER_ID INTEGER REFERENCES USERS(USER_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE
                    )"""

            cursor.execute(query)

            # ----------- ilknur Meray - SHELF TABLE ------------------------------

            query = """CREATE TABLE IF NOT EXISTS SHELF(
                            SHELF_ID SERIAL PRIMARY KEY,
                            SHELF_NAME VARCHAR(50) UNIQUE NOT NULL,
                            IS_MAIN BOOLEAN,
                            BOOK_COUNTER INTEGER DEFAULT 0,
                            SHELF_USER_ID INTEGER REFERENCES USERS(USER_ID)
                            ON DELETE CASCADE ON UPDATE CASCADE
                    )"""

            cursor.execute(query)

            # ----------- ilknur Meray - BOOK TABLE ------------------------------

            query = """CREATE TABLE IF NOT EXISTS BOOK(
                            BOOK_ID SERIAL PRIMARY KEY,
                            BOOK_TITLE VARCHAR(50) NOT NULL,
                            BOOK_COVER VARCHAR(255) NOT NULL,
                            BOOK_WRITER VARCHAR(50) NOT NULL,
                            BOOK_GENRE VARCHAR(50) NOT NULL,
                            DATE_READ DATE NOT NULL,
                            USER_RATE INTEGER DEFAULT 0,
                            BOOK_REVIEW TEXT,
                            BOOK_SHELF_ID INTEGER REFERENCES SHELF(SHELF_ID)
                            ON DELETE CASCADE ON UPDATE CASCADE,
                            BOOK_READER_ID INTEGER REFERENCES USERS(USER_ID)
                            ON DELETE CASCADE ON UPDATE CASCADE
                    )"""

            cursor.execute(query)

            # ----------- ilknur Meray - QUOTE TABLE ------------------------------

            query = """CREATE TABLE IF NOT EXISTS QUOTE(
                            QUOTE_ID SERIAL PRIMARY KEY,
                            QUOTE_CONTENT TEXT NOT NULL,
                            QUOTED_BOOK_ID INTEGER REFERENCES BOOK(BOOK_ID)
                            ON DELETE CASCADE ON UPDATE CASCADE,
                            QUOTE_USER_ID INTEGER REFERENCES USERS(USER_ID)
                            ON DELETE CASCADE ON UPDATE CASCADE
                    )"""

            cursor.execute(query)

            # ------------Nursah Melis Cinar- MESSAGES TABLE----------------------

            query = """CREATE TABLE IF NOT EXISTS MESSAGES(
                        MESSAGE_ID SERIAL PRIMARY KEY,
                        MESSAGE_CONTENT TEXT,
                        FROM_USER_ID INTEGER REFERENCES USERS(USER_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE,
                        TO_USER_ID INTEGER REFERENCES USERS(USER_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE,
                        MESSAGE_DATE DATE NOT NULL
                    )"""

            cursor.execute(query)

            # ------------Nursah Melis Cinar- GROUPS TABLE----------------------

            query = """CREATE TABLE IF NOT EXISTS GROUPS(
                        GROUP_ID SERIAL PRIMARY KEY,
                        GROUP_NAME TEXT NOT NULL,
                        GROUP_PIC varchar(255) NOT NULL,
                        GROUP_DESCRIPTION TEXT NOT NULL
                    )"""

            cursor.execute(query)

            # ------------Nursah Melis Cinar- GROUP_PARTICIPANTS TABLE----------

            query = """CREATE TABLE IF NOT EXISTS GROUP_PARTICIPANTS(
                        GROUP_ID INTEGER REFERENCES GROUPS(GROUP_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE,
                        PARTICIPANT_ID INTEGER REFERENCES USERS(USER_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE
                    )"""

            cursor.execute(query)

            query = """CREATE TABLE IF NOT EXISTS GROUP_KNOT(
                        GROUP_ID INTEGER REFERENCES GROUPS(GROUP_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE,
                        KNOT_ID INTEGER REFERENCES KNOTS(KNOT_ID)
                        ON DELETE CASCADE ON UPDATE CASCADE
                        )"""

            cursor.execute(query)

            connection.commit()
            cursor.close()

.. toctree::

   OzanAta
   TolgaBilbey
   CanAltinigne
   ilknurMeray
   MelisCinar
