---
title: rabbitmq批量删除指定队列
date: 2021-12-23 20:59:21
tags: rabbitmq
categories: 工具
---

#### linux下面删除

首先进入到rabbitmq目录下的sbin目录
方法1：
```
./rabbitmqctl list_queues| grep helloQueue | awk '{print $1}' | xargs -n1 rabbitmqctl delete_queue
```

方法2：
```
./rabbitmqctl list_queues| grep helloQueue | awk '{a[$1]} END {for(i in a){rabbitmqctl delete_queue i;}}'、
```


#### docker下面删除

```
/opt/rabbitmq/sbin/rabbitmqctl list_queues| grep 'udata.climb' | awk '{print $1}' | xargs -n1 rabbitmqctl delete_queue
```


#### windows下面删除

```
./rabbitmqctl.bat list_queues| grep 'udata.climb' | awk '{print $1}' | xargs -n1 ./rabbitmqctl.bat delete_queue
```


#### 通用删除（推荐）
通用删除即通过python脚本删除

> 批量删除带有udata.climb的队列：python3 rabbitmq_delete.py -k 'udata.climb' -d 1
> 批量删除带有udata.climb的交换机：python3 rabbitmq_delete.py -k 'udata.climb' -d 2



```
##### rabbitmq_delete.py
# -*- coding:UTF-8 -*-
from rabbitmq.rabbitmq_api import AdminAPI
import sys
import getopt
 
############################################
# example:
# delete queues: python3 rabbitmq_delete.py  -k 'udata.climb' -d 1
# delete exchanges:  python3 rabbitmq_delete.py  -k 'udata.climb' -d 2
############################################
 
host = '127.0.0.1'
port = '15672'
username = 'rabbitmq'
password = 'Qfka7ls2A40JEoMw'
 
def delete_exchange(key):
    api = AdminAPI(url='http://%s:%s' % (host, port), auth=(username, password))
    exchanges = api.list_exchanges()
    for exchange in exchanges:
        if exchange["name"].find(key) != -1:
            api.delete_exchange_for_vhost(exchange["name"], "/")
 
def delete_quques(key):
    api = AdminAPI(url='http://%s:%s' % (host, port), auth=(username, password))
    queues = api.list_queues()
    for queue in queues:
        if queue["name"].find(key) != -1:
            api.delete_queue(queue["name"], "/")
 
if __name__ == "__main__":
    root_path = ""
    is_rename = True
 
    argv = sys.argv[1:]
    if len(argv) < 1:
        print('rabbitmq_delete.py -k <Key> -d <Type>')
        sys.exit()
 
    # 获取命令行参数
    try:
        opts, args = getopt.getopt(argv, "hk:d:", ["kKey=", "dType="])
    except getopt.GetoptError:
        print('rabbitmq_delete.py -k <Key> -d <Type>')
        sys.exit(2)
 
    for opt, arg in opts:
        if opt == '-h':
            print('rabbitmq_delete.py -k <Key> -d <Type>')
            sys.exit()
        elif opt in ("-k", "--kKey"):
            key = arg
        elif opt in ("-d", "--dType"):
            type = arg
    if type == 1:
        delete_quques(key)
    elif type == 2:
        delete_exchange(key)

##### base.py

import json
import requests
from copy import deepcopy


class Resource(object):
    """
    A base class for API resources
    """

    # """List of allowed methods, allowed values are
    # ['GET', 'PUT', 'POST', 'DELETE']"""
    # ALLOWED_METHODS = []

    def __init__(self, url, auth):
        """
        :param url: The RabbitMQ API url to connect to. This should include the
            protocol and port number.
        :type url: str

        :param auth: The authentication to pass to the request. See
            `Requests' authentication`_ documentation. For the simplest case of
            a username and password, simply pass in a tuple of
            ``('username', 'password')``
        :type auth: Requests auth

        .. _Requests' authentication: http://docs.python-requests.org/en/latest/user/authentication/
        """
        self.url = url.rstrip('/')
        self.auth = auth

        self.headers = {
            'Content-type': 'application/json',
        }

    def _api_get(self, url, **kwargs):
        """
        A convenience wrapper for _get. Adds headers, auth and base url by
        default
        """
        kwargs['url'] = self.url + url
        kwargs['auth'] = self.auth

        headers = deepcopy(self.headers)
        headers.update(kwargs.get('headers', {}))
        kwargs['headers'] = headers
        return self._get(**kwargs)

    def _get(self, *args, **kwargs):
        """
        A wrapper for getting things

        :returns: The response of your get
        :rtype: dict
        """
        response = requests.get(*args, **kwargs)

        response.raise_for_status()

        return response.json()

    def _api_put(self, url, **kwargs):
        """
        A convenience wrapper for _put. Adds headers, auth and base url by
        default
        """
        kwargs['url'] = self.url + url
        kwargs['auth'] = self.auth

        headers = deepcopy(self.headers)
        headers.update(kwargs.get('headers', {}))
        kwargs['headers'] = headers
        self._put(**kwargs)

    def _put(self, *args, **kwargs):
        """
        A wrapper for putting things. It will also json encode your 'data' parameter

        :returns: The response of your put
        :rtype: dict
        """
        if 'data' in kwargs:
            kwargs['data'] = json.dumps(kwargs['data'])
        response = requests.put(*args, **kwargs)
        response.raise_for_status()

    def _api_post(self, url, **kwargs):
        """
        A convenience wrapper for _post. Adds headers, auth and base url by
        default
        """
        kwargs['url'] = self.url + url
        kwargs['auth'] = self.auth

        headers = deepcopy(self.headers)
        headers.update(kwargs.get('headers', {}))
        kwargs['headers'] = headers
        self._post(**kwargs)

    def _post(self, *args, **kwargs):
        """
        A wrapper for posting things. It will also json encode your 'data' parameter

        :returns: The response of your post
        :rtype: dict
        """
        if 'data' in kwargs:
            kwargs['data'] = json.dumps(kwargs['data'])
        response = requests.post(*args, **kwargs)
        response.raise_for_status()

    def _api_delete(self, url, **kwargs):
        """
        A convenience wrapper for _delete. Adds headers, auth and base url by
        default
        """
        kwargs['url'] = self.url + url
        kwargs['auth'] = self.auth

        headers = deepcopy(self.headers)
        headers.update(kwargs.get('headers', {}))
        kwargs['headers'] = headers
        self._delete(**kwargs)

    def _delete(self, *args, **kwargs):
        """
        A wrapper for deleting things

        :returns: The response of your delete
        :rtype: dict
        """
        response = requests.delete(*args, **kwargs)
        response.raise_for_status()

 
#### rabbitmq_api.py

from rabbitmq_admin.base import Resource
from six.moves import urllib


class AdminAPI(Resource):
    """
    The entrypoint for interacting with the RabbitMQ Management HTTP API
    """

    def overview(self):
        """
        Various random bits of information that describe the whole system
        """
        return self._api_get('/api/overview')

    def get_cluster_name(self):
        """
        Name identifying this RabbitMQ cluster.
        """
        return self._get(
            url=self.url + '/api/cluster-name',
            headers=self.headers,
            auth=self.auth
        )

    def list_nodes(self):
        """
        A list of nodes in the RabbitMQ cluster.
        """
        return self._api_get('/api/nodes')

    def get_node(self, name, memory=False, binary=False):
        """
        An individual node in the RabbitMQ cluster. Set "memory=true" to get
        memory statistics, and "binary=true" to get a breakdown of binary
        memory use (may be expensive if there are many small binaries in the
        system).
        """
        return self._api_get(
            url='/api/nodes/{0}'.format(name),
            params=dict(
                binary=binary,
                memory=memory,
            ),
        )

    def list_extensions(self):
        """
        A list of extensions to the management plugin.
        """
        return self._api_get('/api/extensions')

    def get_definitions(self):
        """
        The server definitions - exchanges, queues, bindings, users, virtual
        hosts, permissions and parameters. Everything apart from messages.

        This method can be used for backing up the configuration of a server
        or cluster.
        """
        return self._api_get('/api/definitions')

    def post_definitions(self, data):
        """
        The server definitions - exchanges, queues, bindings, users, virtual
        hosts, permissions and parameters. Everything apart from messages.
        POST to upload an existing set of definitions. Note that:

            - The definitions are merged. Anything already existing on the
              server but not in the uploaded definitions is untouched.
            - Conflicting definitions on immutable objects (exchanges, queues
              and bindings) will cause an error.
            - Conflicting definitions on mutable objects will cause the object
              in the server to be overwritten with the object from the
              definitions.
            - In the event of an error you will be left with a part-applied set
              of definitions.

        This method can be used for restoring the configuration of a server
        or cluster.

        :param data: The definitions for a RabbitMQ server
        :type data: dict
        """
        self._api_post('/api/definitions', data=data)

    def list_connections(self):
        """
        A list of all open connections.
        """
        return self._api_get('/api/connections')

    def get_connection(self, name):
        """
        An individual connection.

        :param name: The connection name
        :type name: str
        """
        return self._api_get('/api/connections/{0}'.format(
            urllib.parse.quote_plus(name)
        ))

    def delete_connection(self, name, reason=None):
        """
        Closes an individual connection. Give an optional reason

        :param name: The connection name
        :type name: str

        :param reason: An option reason why the connection was deleted
        :type reason: str
        """
        headers = {'X-Reason': reason} if reason else {}

        self._api_delete(
            '/api/connections/{0}'.format(
                urllib.parse.quote_plus(name)
            ),
            headers=headers,
        )

    def list_connection_channels(self, name):
        """
        List of all channels for a given connection.

        :param name: The connection name
        :type name: str
        """
        return self._api_get('/api/connections/{0}/channels'.format(
            urllib.parse.quote_plus(name)
        ))

    def list_channels(self):
        """
        A list of all open channels.
        """
        return self._api_get('/api/channels')

    def get_channel(self, name):
        """
        Details about an individual channel.

        :param name: The channel name
        :type name: str
        """
        return self._api_get('/api/channels/{0}'.format(
            urllib.parse.quote_plus(name)
        ))

    def list_consumers(self):
        """
        A list of all consumers.
        """
        return self._api_get('/api/consumers')

    def list_consumers_for_vhost(self, vhost):
        """
        A list of all consumers in a given virtual host.

        :param vhost: The vhost name
        :type vhost: str
        """
        return self._api_get('/api/consumers/{0}'.format(
            urllib.parse.quote_plus(vhost)
        ))

    def list_queues(self):
        """
                A list of all queues.
                """
        return self._api_get('/api/queues')

    def delete_queue(self, queue, vhost, if_unused=False):
        """
        Delete a queue.

        :param name: The vhost name
        :type name: str
        """
        return self._api_delete(
        '/api/queues/{0}/{1}'.format(
            urllib.parse.quote_plus(vhost),
            urllib.parse.quote_plus(queue)),
        params={
            'if-unused': if_unused
        })

    def list_exchanges(self):
        """
        A list of all exchanges.
        """
        return self._api_get('/api/exchanges')

    def list_exchanges_for_vhost(self, vhost):
        """
        A list of all exchanges in a given virtual host.

        :param vhost: The vhost name
        :type vhost: str
        """
        return self._api_get('/api/exchanges/{0}'.format(
            urllib.parse.quote_plus(vhost)
        ))

    def get_exchange_for_vhost(self, exchange, vhost):
        """
        An individual exchange

        :param exchange: The exchange name
        :type exchange: str

        :param vhost: The vhost name
        :type vhost: str
        """
        return self._api_get('/api/exchanges/{0}/{1}'.format(
            urllib.parse.quote_plus(vhost),
            urllib.parse.quote_plus(exchange)
        ))

    def create_exchange_for_vhost(self, exchange, vhost, body):
        """
        Create an individual exchange.
        The body should look like:
        ::

            {
                "type": "direct",
                "auto_delete": false,
                "durable": true,
                "internal": false,
                "arguments": {}
            }

        The type key is mandatory; other keys are optional.

        :param exchange: The exchange name
        :type exchange: str

        :param vhost: The vhost name
        :type vhost: str

        :param body: A body for the exchange.
        :type body: dict
        """
        self._api_put(
            '/api/exchanges/{0}/{1}'.format(
                urllib.parse.quote_plus(vhost),
                urllib.parse.quote_plus(exchange)),
            data=body
        )

    def delete_exchange_for_vhost(self, exchange, vhost, if_unused=False):
        """
        Delete an individual exchange. You can add the parameter
        ``if_unused=True``. This prevents the delete from succeeding if the
        exchange is bound to a queue or as a source to another exchange.

        :param exchange: The exchange name
        :type exchange: str

        :param vhost: The vhost name
        :type vhost: str

        :param if_unused: Set to ``True`` to only delete if it is unused
        :type if_unused: bool
        """
        self._api_delete(
            '/api/exchanges/{0}/{1}'.format(
                urllib.parse.quote_plus(vhost),
                urllib.parse.quote_plus(exchange)),
            params={
                'if-unused': if_unused
            },
        )

    def list_bindings(self):
        """
        A list of all bindings.
        """
        return self._api_get('/api/bindings')

    def list_bindings_for_vhost(self, vhost):
        """
        A list of all bindings in a given virtual host.

        :param vhost: The vhost name
        :type vhost: str
        """
        return self._api_get('/api/bindings/{}'.format(
            urllib.parse.quote_plus(vhost)
        ))

    def list_vhosts(self):
        """
        A list of all vhosts.
        """
        return self._api_get('/api/vhosts')

    def get_vhost(self, name):
        """
        Details about an individual vhost.

        :param name: The vhost name
        :type name: str
        """
        return self._api_get('/api/vhosts/{0}'.format(
            urllib.parse.quote_plus(name)
        ))

    def delete_vhost(self, name):
        """
        Delete a vhost.

        :param name: The vhost name
        :type name: str
        """
        self._api_delete('/api/vhosts/{0}'.format(
            urllib.parse.quote_plus(name)
        ))

    def create_vhost(self, name, tracing=False):
        """
        Create an individual vhost.

        :param name: The vhost name
        :type name: str

        :param tracing: Set to ``True`` to enable tracing
        :type tracing: bool
        """
        data = {'tracing': True} if tracing else {}
        self._api_put(
            '/api/vhosts/{0}'.format(urllib.parse.quote_plus(name)),
            data=data,
        )

    def list_users(self):
        """
        A list of all users.
        """
        return self._api_get('/api/users')

    def get_user(self, name):
        """
        Details about an individual user.

        :param name: The user's name
        :type name: str
        """
        return self._api_get('/api/users/{0}'.format(
            urllib.parse.quote_plus(name)
        ))

    def delete_user(self, name):
        """
        Delete a user.

        :param name: The user's name
        :type name: str
        """
        self._api_delete('/api/users/{0}'.format(
            urllib.parse.quote_plus(name)
        ))

    def create_user(self, name, password, password_hash=None, tags=None):
        """
        Create a user

        :param name: The user's name
        :type name: str
        :param password: The user's password. Set to "" if no password is
            desired. Takes precedence if ``password_hash`` is also set.
        :type password: str
        :param password_hash: An optional password hash for the user.
        :type password_hash: str
        :param tags: A list of tags for the user. Currently recognised tags are
            "administrator", "monitoring" and "management". If no tags are
            supplied, the user will have no permissions.
        :type tags: list of str
        """
        data = {
            'tags': ', '.join(tags or [])
        }
        if password:
            data['password'] = password
        elif password_hash:
            data['password_hash'] = password_hash
        else:
            data['password_hash'] = ""

        self._api_put(
            '/api/users/{0}'.format(urllib.parse.quote_plus(name)),
            data=data,
        )

    def list_user_permissions(self, name):
        """
        A list of all permissions for a given user.

        :param name: The user's name
        :type name: str
        """
        return self._api_get('/api/users/{0}/permissions'.format(
            urllib.parse.quote_plus(name)
        ))

    def whoami(self):
        """
        Details of the currently authenticated user.
        """
        return self._api_get('/api/whoami')

    def list_permissions(self):
        """
        A list of all permissions for all users.
        """
        return self._api_get('/api/permissions')

    def get_user_permission(self, vhost, name):
        """
        An individual permission of a user and virtual host.

        :param vhost: The vhost name
        :type vhost: str

        :param name: The user's name
        :type name: str
        """
        return self._api_get('/api/permissions/{0}/{1}'.format(
            urllib.parse.quote_plus(vhost),
            urllib.parse.quote_plus(name)
        ))

    def delete_user_permission(self, name, vhost):
        """
        Delete an individual permission of a user and virtual host.

        :param name: The user's name
        :type name: str

        :param vhost: The vhost name
        :type vhost: str
        """
        self._api_delete('/api/permissions/{0}/{1}'.format(
            urllib.parse.quote_plus(vhost),
            urllib.parse.quote_plus(name)
        ))

    def create_user_permission(self,
                               name,
                               vhost,
                               configure=None,
                               write=None,
                               read=None):
        """
        Create a user permission
        :param name: The user's name
        :type name: str
        :param vhost: The vhost to assign the permission to
        :type vhost: str

        :param configure: A regex for the user permission. Default is ``.*``
        :type configure: str
        :param write: A regex for the user permission. Default is ``.*``
        :type write: str
        :param read: A regex for the user permission. Default is ``.*``
        :type read: str
        """
        data = {
            'configure': configure or '.*',
            'write': write or '.*',
            'read': read or '.*',
        }
        self._api_put(
            '/api/permissions/{0}/{1}'.format(
                urllib.parse.quote_plus(vhost),
                urllib.parse.quote_plus(name)
            ),
            data=data
        )

    def list_policies(self):
        """
        A list of all policies
        """
        return self._api_get('/api/policies')

    def list_policies_for_vhost(self, vhost):
        """
        A list of all policies for a vhost.
        """
        return self._api_get('/api/policies/{0}'.format(
            urllib.parse.quote_plus(vhost)
        ))

    def get_policy_for_vhost(self, vhost, name):
        """
        Get a specific policy for a vhost.

        :param vhost: The virtual host the policy is for
        :type vhost: str
        :param name: The name of the policy
        :type name: str
        """
        return self._api_get('/api/policies/{0}/{1}'.format(
            urllib.parse.quote_plus(vhost),
            urllib.parse.quote_plus(name),
        ))

    def create_policy_for_vhost(
            self, vhost, name,
            definition,
            pattern=None,
            priority=0,
            apply_to='all'):
        """
        Create a policy for a vhost.

        :param vhost: The virtual host the policy is for
        :type vhost: str
        :param name: The name of the policy
        :type name: str

        :param definition: The definition of the policy. Required
        :type definition: dict
        :param priority: The priority of the policy. Defaults to 0
        :param pattern: The pattern of resource names to apply the policy to
        :type pattern: str
        :type priority: int
        :param apply_to: What resource type to apply the policy to.
            Usually "exchanges", "queues", or "all". Defaults to "all"
        :type apply_to: str

        Example ::

            # Makes all queues and exchanges on vhost "/" highly available
            >>> api.create_policy_for_vhost(
            ... vhost="/",
            ... name="ha-all",
            ... definition={"ha-mode": "all"},
            ... pattern="",
            ... apply_to="all")

        """
        data = {
            "pattern": pattern,
            "definition": definition,
            "priority": priority,
            "apply-to": apply_to
        }
        self._api_put(
            '/api/policies/{0}/{1}'.format(
                urllib.parse.quote_plus(vhost),
                urllib.parse.quote_plus(name),
            ),
            data=data,
        )

    def delete_policy_for_vhost(self, vhost, name):
        """
        Delete a specific policy for a vhost.

        :param vhost: The virtual host of the policy
        :type vhost: str
        :param name: The name of the policy
        :type name: str
        """
        self._api_delete('/api/policies/{0}/{1}/'.format(
            urllib.parse.quote_plus(vhost),
            urllib.parse.quote_plus(name),
        ))

    def is_vhost_alive(self, vhost):
        """
        Declares a test queue, then publishes and consumes a message.
        Intended for use by monitoring tools.

        :param vhost: The vhost name to check
        :type vhost: str
        """
        return self._api_get('/api/aliveness-test/{0}'.format(
            urllib.parse.quote_plus(vhost)
        ))
```
