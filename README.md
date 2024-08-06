# GeoServer Installation and Shapefile Handling Guide

## Introduction

GeoServer is a JAVA-based application developed to ease the styling and sharing of geospatial data using Open Source technology. It follows the standards of the Open Geospatial Consortium (OGC) and thus has wide application in a variety of industries. This guide provides instructions on installing GeoServer and working with shapefiles in QGIS, PostgreSQL/PostGIS, and GeoServer.

## Installation Guide

### Installing GeoServer on Linux

Installing GeoServer in a GUI-based system such as Windows, Mac, or even Ubuntu GUI is straightforward. However, if you only have access to an SSH terminal, follow the steps below:

#### Step 1: Check/Install JAVA 17

GeoServer currently supports JAVA-JRE-17. Install it using the following commands:

```bash
sudo apt-get update
sudo apt-get install openjdk-17-jdk
sudo apt-get install openjdk-17-jre
```

Verify the installation by typing `java -version`.

#### Step 2: Installation of Postgres (Optional)

If you wish to use a database with GeoServer, it is recommended to install PostgreSQL. PostGIS is likely already installed if you're running PostgreSQL as a service from a cloud provider like DigitalOcean. To enable PostGIS, you can:

1. Connect to your database as the postgres user or another super-user account.
2. Run the command: `CREATE EXTENSION postgis`

#### Step 3: Downloading GeoServer

Downloading GeoServer without a GUI requires manual downloading and installation. Visit the GeoServer build page [here](https://build.geoserver.org/geoserver/) to select the desired version. For example:

```bash
cd /usr/share
mkdir geoserver
cd geoserver
wget https://build.geoserver.org/geoserver/main/geoserver-main-latest-bin.zip
```

Unzip the downloaded file:

```bash
unzip geoserver-main-latest-bin.zip
```

Set up a variable:

```bash
echo "export GEOSERVER_HOME=/usr/share/geoserver" >> ~/.profile
. ~/.profile
```

Ensure the correct ownership:

```bash
sudo chown -R USER_NAME /usr/share/geoserver/
```

#### Step 4: Starting GeoServer

Navigate to the bin directory and run `startup.sh`:

```bash
cd bin
sh startup.sh
```

Once GeoServer is up and running, access it through a web browser using the designated port (usually 8080).

#### Step 5: Configuring GeoServer for Usage and Running as a Service

To make GeoServer accessible publicly and resolve CORS issues, edit the `web.xml` file located at `geoserver/webapps/geoserver/WEB-INF`. Use a text editor like Vim:

```bash
sudo apt install vim
sudo vim web.xml
```

Add the following lines to the `web.xml` file, replacing the URL with your chosen URL:

```xml
<context-param>
  <param-name>PROXY_BASE_URL</param-name>
  <param-value>https://map.homeful.ph/geoserver</param-value>
</context-param>

<context-param>
  <param-name>GEOSERVER_CSRF_WHITELIST</param-name>
  <param-value>homeful.ph,map.homeful.ph,map.homeful.ph/geoserver</param-value>
</context-param>
```

Uncomment the CORS-related lines and save the changes. Then restart GeoServer.

To run GeoServer as a service, follow the instructions provided in the [official documentation](https://docs.geoserver.org/2.21.x/en/user/production/linuxscript.html).

#### Step 6: NGINX Configuration (Optional)

If you're using NGINX as a reverse proxy to serve GeoServer, you'll need to configure NGINX to pass requests to GeoServer. Add the following configuration to your NGINX configuration file (usually located at `/etc/nginx/nginx.conf` or in a site-specific configuration file):

```nginx
location /geoserver {
  proxy_pass http://localhost:8080/geoserver;
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_redirect ~http://[^/]+(/.)$ $1;
}
```

Replace `/geoserver` with the URL path you want to use for GeoServer. After adding the configuration, restart NGINX for the changes to take effect.

#### Step 7: Ensure GeoServer is Running Properly

Remove the Marlin renderer library jar file to ensure proper functioning:

```bash
rm webapps/geoserver/WEB-INF/lib/marlin-0.9.3.jar
```

#### Step 8: Access GeoServer

You can access GeoServer using the default credentials:
- Username: admin
- Password: geoserver

### Installing GeoServer on Windows

#### Step 1: Install JAVA 17

1. Download and install the Java Development Kit (JDK) version 17 from the [Oracle website](https://www.oracle.com/java/technologies/javase-jdk17-downloads.html).
2. During installation, ensure that the JAVA_HOME environment variable is set.

#### Step 2: Install PostgreSQL (Optional)

1. Download and install PostgreSQL from the [official website](https://www.postgresql.org/download/windows/).
2. During installation, choose to include the PostGIS extension if you plan to use it with GeoServer.

#### Step 3: Download and Install GeoServer

1. Download the latest stable GeoServer Windows installer from the [GeoServer website](http://geoserver.org/download/).
2. Run the installer and follow the on-screen instructions.
3. Choose the directory where you want GeoServer to be installed.

#### Step 4: Start GeoServer

1. After installation, start GeoServer by navigating to the Start Menu and selecting **Start GeoServer**.
2. GeoServer will start and be accessible at `http://localhost:8080/geoserver` by default.

#### Step 5: Configure GeoServer for Usage

1. Once GeoServer is running, open a web browser and navigate to `http://localhost:8080/geoserver`.
2. Log in using the default credentials:
   - Username: admin
   - Password: geoserver

### Troubleshooting

If you encounter errors while accessing GeoServer, double-check the JAVA installation, the GeoServer logs, and ensure that the necessary environment variables are correctly set.

## Loading a Shapefile to QGIS, Importing to PostgreSQL/PostGIS, and Publishing in GeoServer

### **1. Loading a Shapefile into QGIS**

#### **1.1. Open QGIS**
1. Download and install QGIS if you haven't already. You can get it from the [QGIS official website](https://qgis.org/).
2. Launch QGIS.

#### **1.2. Load the Shapefile**
1. Go to the **Layers** panel on the left side of the screen.
2. Click on the **Open Data Source Manager** button (or press `Ctrl + L`).
3. In the Data Source Manager, select **Vector** under the **Source Type**.
4. Under the **Vector Dataset(s)** section, click the **…** (Browse) button to navigate to the directory containing your shapefile (`.shp`).
5. Select the shapefile and click **Open**.
6. Click **Add** and then **Close** the Data Source Manager.

   Your shapefile should now be visible in the QGIS interface.

### **2. Importing the Shapefile to PostgreSQL/PostGIS**

#### **2.1. Prepare PostgreSQL/PostGIS**
1. Ensure you have PostgreSQL installed with the PostGIS extension enabled.
2. Open **pgAdmin** or your preferred PostgreSQL client and connect to your PostgreSQL server.
3. Create a database (if not already existing) by executing:
   ```sql
   CREATE DATABASE your_database_name;
   ```
4. Enable PostGIS on your database:
   ```sql
   CREATE EXTENSION postgis;
   ```

#### **2.2. Import the Shapefile Using DB Manager**
1. Go back to QGIS.
2. In the **Browser Panel**, locate the **DB Manager** (usually under the **Database** menu).
3. Open **DB Manager** and connect to your PostgreSQL/PostGIS database.
4. In DB Manager, click on **Import Layer/File**.
5. Choose your shapefile under the **Input File** option.
6. Select the appropriate database from the drop-down list.
7. Configure any additional options such as the **Schema**, **Table name**, and **Geometry** type.
8. Click **OK** to start the import process.

   The shapefile data should now be imported into your PostgreSQL/PostGIS database as a table.

### **3. Publishing the Layer/Table in GeoServer**

#### **3.1. Access GeoServer**
1. Make sure GeoServer is installed and running. You can download it from the [GeoServer website](http://geoserver.org/).
2. Open your web browser and navigate to `http://localhost:8080/geoserver` (assuming default port settings).

#### **3.2. Add a New PostGIS Data Store**
1. Log in to GeoServer using your credentials.
2. Navigate to **Data** → **Stores**.
3. Click on **Add new Store**.
4. Choose **PostGIS** as the
5. Fill in the necessary connection details:
   - **Database**: your PostgreSQL/PostGIS database name
   - **User**: your PostgreSQL user
   - **Password**: your PostgreSQL user password
   - **Host**: your PostgreSQL server address (e.g., `localhost`)
   - **Port**: usually `5432` by default
6. Click **Save**.

### **3.3. Publish the Layer**
1. After saving the store, you’ll be prompted to create a new layer from the tables in your PostGIS database.
2. Select the table corresponding to the shapefile you imported.
3. Click on **Publish**.
4. Configure the **Layer settings** (like SRS, Bounding Box, etc.) as needed.
5. Click **Save** to publish the layer.

### **3.4. Preview the Published Layer**
1. Go to **Layer Preview** under the **Data** menu.
2. Find your newly published layer in the list and click on the **OpenLayers** link or any other format to view it.

---

This guide should help you get your shapefile loaded into QGIS, imported into PostgreSQL/PostGIS, and published as a layer in GeoServer. Let me know if you need any more details or if there's anything else you'd like to cover!
