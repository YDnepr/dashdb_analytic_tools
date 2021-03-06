# (c) Copyright IBM Corporation 2016
# LICENSE: BSD-3, https://opensource.org/licenses/BSD-3-Clause

FROM jupyter/base-notebook
# tested with version:
# FROM jupyter/base-notebook:c411f52fcc93
USER root

# sort packages alphabetically following docker convention
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    default-jre-headless \
    unzip \
    zip

# sbt is not in repo for Trusty
RUN cd /tmp && \
	wget -nv https://bintray.com/artifact/download/sbt/debian/sbt-0.13.11.deb && \
	dpkg -i sbt-0.13.11.deb && \
	rm *.deb

# don't need to download a debian packaged scala interpreter; sbt will get all we need
# wget -nv www.scala-lang.org/files/archive/scala-2.11.8.deb

# first start of sbt downloads maven dependencies -- do that early in the docker buid process
# need to define correct scalaVersion and dependencies for Spark before we start downloading
COPY build.sbt.template tmp/build.sbt
COPY plugins.sbt tmp/project/
RUN chown -R $NB_USER tmp
USER $NB_USER
RUN cd tmp &&  \
	mkdir -p src/main/scala && \
	echo "object Dummy {}" > src/main/scala/Dummy.scala && \
	echo "Note: downloading sbt may seem to be stuck for a few minutes, be patient" && \
	sbt assembly && \
	cd .. && rm -rf tmp

# set up jupyter environment
# widgetsnbextension needed to avoid warning during notebook start
RUN pip install jupyter_cms widgetsnbextension && \
	jupyter cms quick-setup --sys-prefix


USER root

ENV PROJECTS_DIR /home/$NB_USER/projects

# copy other files late in the build to avoid cache busting
COPY toree-assembly-0.2.0.dev1-incubating-SNAPSHOT.jar /usr/local/lib/toree.jar
COPY build.sbt.template build.sh $PROJECTS_DIR/
COPY plugins.sbt $PROJECTS_DIR/sparkapp/project/
COPY kernel-scala.json /home/$NB_USER/.local/share/jupyter/kernels/idax-scala/kernel.json
COPY kernel-python.json /home/$NB_USER/.local/share/jupyter/kernels/idax-python/kernel.json
COPY verify-sparkenv.py upload-sparkapp.py patch-ui.py run-toree.py launch-with-idax.sh /usr/local/bin/
COPY Spark_KMeansSample.ipynb Tornado_Clustering.ipynb ./

# copy jupyter CMS extensions
ADD src /src

# fix permissions if git repo was checked out as root
# fix ownership for all files copied into homedir
RUN chmod -R a+rX /src && \
	chmod a+rx /usr/local/bin/* /usr/local/lib/* && \
	chown -R $NB_USER /home/$NB_USER

USER $NB_USER

# set up template project
RUN bash -c "mkdir -p $PROJECTS_DIR/sparkapp/src/main/{java,scala,resources}" && \
	cp $PROJECTS_DIR/build.sh $PROJECTS_DIR/sparkapp/

# install and enable spark app bundler
RUN pip install /src/sparkapp_bundler && \
	jupyter serverextension enable --py jupyter_cms_sparkapp --sys-prefix && \
	jupyter bundler enable --py jupyter_cms_sparkapp --sys-prefix


# put our setup on top of the base image startup script
CMD ["launch-with-idax.sh"]
