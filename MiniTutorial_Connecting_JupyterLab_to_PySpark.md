<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tutorial: Building a Containerized Data Lakehouse</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Fira+Code&display=swap" rel="stylesheet">
    <!-- 
        Chosen Palette: Warm Neutrals (Stone and Amber)
        Application Structure Plan: A vertical, single-page timeline/stepper layout. This structure is ideal for a tutorial as it guides the user through a linear process from start to finish. Each major phase is a distinct section with a clear number, title, and description. The final "Lessons Learned" section is presented as a summary card to emphasize its importance. This is more intuitive for a step-by-step guide than tabbed navigation.
        Visualization & Content Choices: 
        - Report Info: The full tutorial text from the user's Markdown file.
        - Goal: To present a complex technical tutorial in an easy-to-follow, visually appealing, and interactive format.
        - Viz/Presentation Method: The core is structured HTML with Tailwind CSS. Code blocks are a central element.
        - Interaction: One-click "Copy" buttons for all code blocks to reduce user error and friction. This is the most critical interaction for a technical tutorial.
        - Justification: The stepper format with copyable code blocks is a best-in-class user experience for technical documentation.
        - Library/Method: Vanilla JS for the copy-to-clipboard functionality.
        - CONFIRMATION: NO SVG graphics used. NO Mermaid JS used.
    -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f5f5f4; /* stone-100 */
        }
        .code-block {
            font-family: 'Fira Code', monospace;
            position: relative;
            background-color: #292524; /* stone-800 */
            color: #e7e5e4; /* stone-200 */
            border-radius: 0.5rem;
            padding: 1.5rem;
            overflow-x: auto;
        }
        .code-block pre {
            margin: 0;
            white-space: pre;
        }
        .copy-btn {
            position: absolute;
            top: 0.75rem;
            right: 0.75rem;
            background-color: #57534e; /* stone-600 */
            color: #e7e5e4; /* stone-200 */
            border: none;
            padding: 0.5rem 0.75rem;
            border-radius: 0.375rem;
            cursor: pointer;
            font-size: 0.875rem;
            transition: background-color 0.2s;
        }
        .copy-btn:hover {
            background-color: #78716c; /* stone-500 */
        }
        .copy-btn-copied {
            background-color: #16a34a;
        }
        .step-number {
            background-color: #f59e0b; /* amber-500 */
            color: white;
            width: 3rem;
            height: 3rem;
            border-radius: 9999px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 700;
            font-size: 1.25rem;
            flex-shrink: 0;
        }
        .timeline-line {
            width: 2px;
            background-color: #d6d3d1; /* stone-300 */
        }
    </style>
</head>
<body class="antialiased">

    <div class="max-w-4xl mx-auto p-4 sm:p-6 lg:p-8">
        <!-- Header -->
        <header class="text-center mb-12">
            <h1 class="text-3xl sm:text-4xl font-bold text-stone-900">Tutorial: Building a Containerized Data Lakehouse</h1>
            <p class="mt-4 text-lg text-stone-600">The complete guide to packaging MinIO, PostgreSQL, Spark, and Jupyter into a professional, portable system using Docker, including the lessons learned from troubleshooting.</p>
        </header>

        <!-- Main Tutorial Content -->
        <div class="space-y-12">

            <!-- Phase 1 -->
            <div class="flex">
                <div class="flex flex-col items-center mr-6">
                    <div class="step-number">1</div>
                    <div class="timeline-line h-full"></div>
                </div>
                <div class="w-full">
                    <h2 class="text-2xl font-bold text-stone-800 mb-2">Phase 1: Project Structure</h2>
                    <p class="text-stone-600 mb-6">A clean project structure is essential for Docker. This ensures our build process is self-contained and reproducible.</p>
                    <div class="space-y-4">
                        <div>
                            <h3 class="font-semibold mb-2">Create Project Directories</h3>
                            <div class="code-block">
                                <button class="copy-btn">Copy</button>
                                <pre><code class="language-bash">mkdir -p ~/docker-lakehouse/spark
mkdir ~/docker-lakehouse/notebooks</code></pre>
                            </div>
                        </div>
                        <div>
                            <h3 class="font-semibold mb-2">Pre-download All Dependencies</h3>
                            <p class="text-stone-600 text-sm mb-2">To make our Docker build robust and immune to network errors, we download all necessary files into the <code class="text-xs bg-stone-200 rounded px-1">spark</code> directory. This is our "build context".</p>
                            <div class="code-block">
                                <button class="copy-btn">Copy</button>
                                <pre><code class="language-bash"># Navigate to the spark directory
cd ~/docker-lakehouse/spark

# Download Spark and Hadoop
wget https://archive.apache.org/dist/spark/spark-3.5.1/spark-3.5.1-bin-hadoop3.tgz
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz

# Download all required JARs
wget https://repo.maven.apache.org/maven2/org/apache/iceberg/iceberg-spark-runtime-3.5/1.5.2/iceberg-spark-runtime-3.5-1.5.2.jar
wget https://jdbc.postgresql.org/download/postgresql-42.7.3.jar
wget https://repo.maven.apache.org/maven2/software/amazon/awssdk/bundle/2.25.30/bundle-2.25.30.jar
wget https://repo.maven.apache.org/maven2/org/apache/hadoop/hadoop-aws/3.3.4/hadoop-aws-3.3.4.jar
wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.12.262/aws-java-sdk-bundle-1.12.262.jar</code></pre>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Phase 2 -->
            <div class="flex">
                <div class="flex flex-col items-center mr-6">
                    <div class="step-number">2</div>
                    <div class="timeline-line h-full"></div>
                </div>
                <div class="w-full">
                    <h2 class="text-2xl font-bold text-stone-800 mb-2">Phase 2: The `Dockerfile` (The Recipe)</h2>
                    <p class="text-stone-600 mb-6">This file, located at <code class="text-sm bg-stone-200 rounded px-1">~/docker-lakehouse/spark/Dockerfile</code>, is the recipe for our custom Spark+Jupyter container.</p>
                    <div class="code-block">
                        <button class="copy-btn">Copy</button>
                        <pre><code class="language-dockerfile"># Start from a base Jupyter image
FROM jupyter/base-notebook:latest

# Switch to root to install software
USER root

# Install Java
RUN apt-get update && \\
    apt-get install -y openjdk-17-jdk && \\
    apt-get clean && \\
    rm -rf /var/lib/apt/lists/*

# Copy archives into the image
COPY spark-3.5.1-bin-hadoop3.tgz /opt/
COPY hadoop-3.3.6.tar.gz /opt/

# Set all necessary environment variables
ENV SPARK_HOME=/opt/spark-3.5.1-bin-hadoop3
ENV HADOOP_HOME=/opt/hadoop-3.3.6
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
ENV PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin:$HADOOP_HOME/bin
ENV LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH
ENV PYTHONPATH=$SPARK_HOME/python/:$PYTHONPATH

# Extract archives
RUN tar -xzf /opt/spark-3.5.1-bin-hadoop3.tgz -C /opt/ && \\
    tar -xzf /opt/hadoop-3.3.6.tar.gz -C /opt/ && \\
    rm /opt/*.tgz

# Copy pre-downloaded JARs into Spark's jars directory
COPY *.jar $SPARK_HOME/jars/

# Install the py4j library for Python-to-Java communication
RUN pip install py4j

# Switch back to the non-root user
USER $NB_UID

# Set the working directory
WORKDIR /home/jovyan/work</code></pre>
                    </div>
                </div>
            </div>

            <!-- Phase 3 -->
            <div class="flex">
                <div class="flex flex-col items-center mr-6">
                    <div class="step-number">3</div>
                    <div class="timeline-line h-full"></div>
                </div>
                <div class="w-full">
                    <h2 class="text-2xl font-bold text-stone-800 mb-2">Phase 3: The `docker-compose.yml` (The Conductor)</h2>
                    <p class="text-stone-600 mb-6">This file, at <code class="text-sm bg-stone-200 rounded px-1">~/docker-lakehouse/docker-compose.yml</code>, defines and connects all our services.</p>
                    <div class="code-block">
                        <button class="copy-btn">Copy</button>
                        <pre><code class="language-yaml">version: "3.8"
services:
  postgres-catalog:
    image: postgres:14
    container_name: postgres-catalog
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=iceberg
      - POSTGRES_PASSWORD=iceberg
      - POSTGRES_DB=iceberg_catalog
    ports:
      - "5432:5432"
    networks:
      - lakehouse-net

  minio:
    image: minio/minio:latest
    container_name: minio
    ports:
      - "9000:9000"
      - "9090:9090"
    volumes:
      - minio-data:/data
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
      - MINIO_DEFAULT_BUCKETS=datalake
    command: server /data --console-address ":9090"
    networks:
      - lakehouse-net

  spark-jupyter:
    build: ./spark
    container_name: spark-jupyter
    ports:
      - "8888:8888"
      - "4040:4040"
    volumes:
      - ./notebooks:/home/jovyan/work
    networks:
      - lakehouse-net
volumes:
  postgres-data:
  minio-data:
networks:
  lakehouse-net:
    driver: bridge</code></pre>
                    </div>
                </div>
            </div>

            <!-- Phase 4 -->
            <div class="flex">
                <div class="flex flex-col items-center mr-6">
                    <div class="step-number">4</div>
                    <div class="timeline-line h-full"></div>
                </div>
                <div class="w-full">
                    <h2 class="text-2xl font-bold text-stone-800 mb-2">Phase 4: The PySpark Notebook Code</h2>
                    <p class="text-stone-600 mb-6">This is the final, correct code to run in the first cell of every new Jupyter notebook. It relies on Spark's own package manager to ensure all dependencies are loaded correctly, which is the most robust method.</p>
                    <div class="code-block">
                        <button class="copy-btn">Copy</button>
                        <pre><code class="language-python">from pyspark.sql import SparkSession

# This is the final and most reliable configuration.
# It uses Spark's built-in package manager to download and correctly
# configure the entire dependency tree for all required libraries.
spark = SparkSession.builder \\
    .appName("DockerIcebergFinal") \\
    .config("spark.jars.packages", "org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.5.2,org.postgresql:postgresql:42.7.3,org.apache.hadoop:hadoop-aws:3.3.4,com.amazonaws:aws-java-sdk-bundle:1.12.262") \\
    .config("spark.sql.extensions", "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions") \\
    .config("spark.sql.catalog.local", "org.apache.iceberg.spark.SparkCatalog") \\
    .config("spark.sql.catalog.local.type", "jdbc") \\
    .config("spark.sql.catalog.local.uri", "jdbc:postgresql://postgres-catalog:5432/iceberg_catalog") \\
    .config("spark.sql.catalog.local.jdbc.user", "iceberg") \\
    .config("spark.sql.catalog.local.jdbc.password", "iceberg") \\
    .config("spark.sql.catalog.local.warehouse", "s3a://datalake/warehouse") \\
    .config("spark.hadoop.fs.s3a.endpoint", "http://minio:9000") \\
    .config("spark.hadoop.fs.s3a.access.key", "minioadmin") \\
    .config("spark.hadoop.fs.s3a.secret.key", "minioadmin") \\
    .config("spark.hadoop.fs.s3a.path.style.access", "true") \\
    .config("spark.hadoop.fs.s3a.connection.ssl.enabled", "false") \\
    .config("spark.hadoop.fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem") \\
    .getOrCreate()

print("SparkSession created successfully!")</code></pre>
                    </div>
                </div>
            </div>

            <!-- Lessons Learned -->
            <div class="flex">
                <div class="flex flex-col items-center mr-6">
                    <div class="step-number">★</div>
                </div>
                <div class="w-full">
                    <h2 class="text-2xl font-bold text-stone-800 mb-2">Lessons Learned: A Summary of Our Battle</h2>
                    <p class="text-stone-600 mb-6">Our journey to containerize this project was difficult but taught us several critical lessons about Spark and Docker.</p>
                    <div class="bg-amber-50 border-l-4 border-amber-500 p-6 rounded-r-lg space-y-4">
                        <div>
                            <h4 class="font-bold text-amber-900">Mistake 1: Managing JARs Manually</h4>
                            <p class="text-amber-800">Our biggest struggle was trying to avoid Spark's package manager to speed up startup. This led to a series of `ClassNotFoundException` errors.</p>
                            <p class="font-semibold text-amber-900 mt-1">Lesson: For complex dependencies, Spark's own package manager (`spark.jars.packages`) is the most reliable method. The one-time download cost is worth the stability.</p>
                        </div>
                        <div>
                            <h4 class="font-bold text-amber-900">Mistake 2: Using `findspark` in Docker</h4>
                            <p class="text-amber-800">We initially used `findspark` for our manual setup. In a clean, containerized environment, this became an anti-pattern that caused conflicts.</p>
                            <p class="font-semibold text-amber-900 mt-1">Lesson: `findspark` is for messy, local setups. In a clean Docker image, environment variables (`SPARK_HOME`, `PYTHONPATH`) are the correct way to make PySpark available.</p>
                        </div>
                        <div>
                            <h4 class="font-bold text-amber-900">Mistake 3: Forgetting Python Dependencies</h4>
                            <p class="text-amber-800">Our notebook failed with `ModuleNotFoundError: No module named 'py4j'`. </p>
                            <p class="font-semibold text-amber-900 mt-1">Lesson: A Docker image is an isolated system. You must explicitly install every dependency, including Python packages (`pip install py4j`).</p>
                        </div>
                        <div>
                            <h4 class="font-bold text-amber-900">Mistake 4: Re-using a Broken SparkSession</h4>
                            <p class="text-amber-800">We frequently saw errors return after restarting a cell. This was caused by the warning: `Using an existing Spark session`.</p>
                            <p class="font-semibold text-amber-900 mt-1">Lesson: If a SparkSession is created incorrectly, it can get stuck in a broken state. Always use **Kernel > Restart Kernel** in Jupyter to ensure a completely clean start.</p>
                        </div>
                    </div>
                </div>
            </div>

        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function () {
            const copyButtons = document.querySelectorAll('.copy-btn');
            copyButtons.forEach(button => {
                button.addEventListener('click', () => {
                    const codeBlock = button.closest('.code-block');
                    const code = codeBlock.querySelector('code').innerText;
                    navigator.clipboard.writeText(code).then(() => {
                        button.textContent = 'Copied!';
                        button.classList.add('copy-btn-copied');
                        setTimeout(() => {
                            button.textContent = 'Copy';
                            button.classList.remove('copy-btn-copied');
                        }, 2000);
                    }).catch(err => console.error('Failed to copy text: ', err));
                });
            });
        });
    </script>

</body>
</html>
