.. _storage:

Storage adapter implementation
------------------------------

Currently, there are available prepared database adapters, see `Selinonlib <https://github.com/selinon/selinonlib>`_ module. In order to use these storages, you have to manually install database adapters as they are not explicitly included by requirements.

  * `SqlStorage` - SQLAlchemy adapter for SQL databases

    * necessary dependencies for this storage adapter:

      `pip3 install SQLAlchemy SQLAlchemy-Utils`

    * configuration example:

      .. code-block:: yaml

        storages:
          - name: 'MySqlStorage'
            classname: 'SqlStorage'
            import: 'selinon.storage'
            configuration:
              connection_string: 'postgres://postgres:postgres@postgres:5432/postgres'
              encoding: 'utf-8'
            echo: false

  * `RedisStorage` - Redis database adapter

    * necessary dependencies for this storage adapter:

      `pip3 install redis`

    * configuration example:

      .. code-block:: yaml

        storages:
          - name: 'MyRedisStorage'
            classname: 'RedisStorage'
            import: 'selinon.storage'
            configuration:
              host: 'redishost'
              port: 6379
              db: 0
              password: 'secretpassword'
              charset: 'utf-8'
              host: 'mongohost'
            port: 27017

  * `MongoStorage` - MongoDB database adapter

    * necessary dependencies for this storage adapter:

      `pip3 install pymongo`

    * configuration example:

      .. code-block:: yaml

        storages:
          - name: 'MyMongoStorage'
            classname: 'MongoStorage'
            import: 'selinon.storage'
            configuration:
              db_name: 'database_name'
              collection_name: 'collection_name'
              host: 'mongohost'
            port: 27017

  * `S3` - AWS S3 database adapter

    * necessary dependencies for this storage adapter:

      `pip3 install boto3`

    * configuration example:

    .. code-block:: yaml

      storages:
        - name: 'MyS3Storage'
          classname: 'S3Storage'
          import: 'selinon.storage'
          configuration:
            bucket: 'my-bucket-name'
            aws_access_key_id: 'AAAAAAAAAAAAAAAAAAAA'
            aws_secret_access_key: 'BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB'
            region_name: 'us-east-1'


Using a custom storage adapter
##############################

You can define your own storage by inheriting from :class:`DataStorage <selinon.dataStorage.DataStorage>` abstract class:

::

  from selinon import DataStorage

  class MyStorage(DataStorage):
      def __init__(self, host, port):
          # arguments from YAML file are pasased to constructor as key-value arguments
          pass

      def is_connected():
          # predicate used to check connection
          return False

      def connect():
          # define how to connect based on your configuration
          pass

      def disconnect():
          # define how to disconnect from storage
          pass

      def retrieve(self, flow_name, task_name, task_id):
          # define how to retrieve results
          pass

      def store(self, flow_name, task_name, task_id, result):
          # define how to store results
          pass

      def store_error(self, node_args, flow_name, task_name, task_id, exc_info):
          # optionally define how to track errors/task failures if you need to
          pass

And pass this storage to Selinon in your YAML configuration:

.. code-block:: yaml

  storages:
    # from myapp.storages import MyStorage
    - name: 'MyStorage'
      import: 'myapp.storages'
      configuration:
        host: 'localhost'
        port: '5432'

If you create an adapter for some well known storage and you feel that your adapter is generic enough, feel free to share it with community by opening a pull request!

Database connection pool
########################

Each worker is trying to be efficient when it comes to number of connections to a database. There is held only one instance of :class:`DataStorage <selinon.dataStorage.DataStorage>` class per whole worker. Selinon transparently takes care of concurrent-safety when calling methods of :class:`DataStorage <selinon.dataStorage.DataStorage>` if you plan to run your worker with concurrency level higher than one.


.. note::

  You can also simply share connection across multiple :class:`DataStorage <selinon.dataStorage.DataStorage>` classes in inheritance hierarchy and reuse already defined connections. You can also do storage aliasing as described in :ref:`practices`.

If you would like to request some storage from your configuration, you can request storage adapter from Selinon :class:`StoragePool <selinon.storagePool>`:

.. code-block:: python

   from selinon import StoragePool

   # Name of storage was set to MyMongoStorage in nodes.yaml configuration file (section storages).
   mongo = StoragePool.get_connected_storage('MyMongoStorage')

Selinon will transparently take care of instantiation, connection and sharing connection pool across the whole process. Check out other useful methods of :class:`StoragePool <selinon.storagePool>`.
