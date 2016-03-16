New functions for notebook extensions
=====================================

_get_nbextension_metadata(package)
----------------------------------
def _get_nbextension_metadata(package):
    """Get the list of nbextension paths associated with a python package.
    Returns a tuple of (the module,             [{
        'section': 'notebook',
        'src': 'mockextension',
        'dest': '_mockdestination',
        'require': '_mockdestination/index'
    }])
    Parameters
    ----------
    package : str
        Importable Python package (no dotted-notation!) exposing the
        magic-named `_jupyter_nbextension_paths` function
    """

_get_server_extension_metadata(package)
---------------------------------------
def _get_server_extension_metadata(package):
    """Load server extension metadata from a package's magic-named path.
    Returns a tuple of (
        the package as loaded
        a list of server extension specs: [
            {
                "module": "mockextension"
            }
        ]
    )
    Parameters
    ----------
    package : str
        Importable Python package (no dotted-notation!) exposing the
        magic-named `_jupyter_server_extension_paths` function
    """
	
	
I believe that the two functions above replaced:
 
_get_nbextension_paths()
_get_serverextension_paths()


This should be jupyter serverextension enable --py my_fancy_module [--sys-prefix|--user] I think. 
There is no serverextension install subcommand according to the CLI.


+    def toggle_nbextension_python(self, package):
 +        """Toggle some extensions in an importable Python package.
 +
 +        Returns a list of booleans indicating whether the state was changed as
 +        requested.
 +
 +        Parameters
 +        ----------
 +        package : str
 +            Importable Python package (no dotted-notation!) exposing the
 +            magic-named `_jupyter_nbextension_paths` function
 +        """
 
 def _get_nbextension_metadata(package):
 
 package : str
 +        Importable Python package (no dotted-notation!) exposing the
 +        magic-named `_jupyter_nbextension_paths` function
 +    """
 +    m = __import__(package)
 +    if not hasattr(m, '_jupyter_nbextension_paths'):
 +        raise KeyError('The Python package {} is not a valid nbextension'.format(package))
 +    nbexts = m._jupyter_nbextension_paths()
 +    return m, nbexts
 
 def toggle_serverextension_python(import_name, enabled=None, parent=None,
 +                                  user=False, sys_prefix=False, logger=None):
 +    """Toggle a server extension.
 +
 +    By default, toggles the extension in the system-wide Jupyter configuration
 +    location (e.g. /usr/local/etc/jupyter).
 +
 +    Parameters
 +    ----------
 +
 +    import_name : str
 +        Importable Python module (dotted-notation) exposing the magic-named
 +        `load_jupyter_server_extension` function
 +    enabled : bool [default: None]
 +        Toggle state for the extension.  Set to None to toggle, True to enable,
 +        and False to disable the extension.
 +    parent : Configurable [default: None]
 +    user : bool [default: False]
 +        Toggle in the user's configuration location (e.g. ~/.jupyter).
 +    sys_prefix : bool [default: False]
 +        Toggle in the current Python environment's configuration location
 +        (e.g. ~/.envs/my-env/etc/jupyter).
 +    logger : Jupyter logger [optional]
 +        Logger instance to use
 +    """
 +    config_dir = _get_config_dir(user=user, sys_prefix=sys_prefix)
 +    cm = BaseJSONConfigManager(parent=parent, config_dir=config_dir)
 +    cfg = cm.get("jupyter_notebook_config")
 +    server_extensions = (
 +        cfg.setdefault("NotebookApp", {})
 +        .setdefault("nbserver_extensions", {})
 +    )
 +
 +    old_enabled = server_extensions.get(import_name, None)
 +    new_enabled = enabled if enabled is not None else not old_enabled
 +
 +    if logger:
 +        if new_enabled:
 +            logger.info("Enabling: %s" % (import_name))
 +        else:
 +            logger.info("Disabling: %s" % (import_name))
 +
 +    server_extensions[import_name] = new_enabled
 +
 +    if logger:
 +        logger.info("- Writing config: {}".format(config_dir))
 +
 +    cm.update("jupyter_notebook_config", cfg)
 +
 +    if new_enabled:
 +        validate_serverextension(import_name, logger)
 
 def toggle_server_extension_python(self, package):
+        """Change the status of some server extensions in a Python package.
+
+        Uses the value of `self._toggle_value`.
+
+        Parameters
+        ---------
+
+        package : str
+            Importable Python package (no dotted-notation!) exposing the
+            magic-named `_jupyter_server_extension_paths` function
+        """
+        m, server_exts = _get_server_extension_metadata(package)
+        for server_ext in server_exts:
+            module = server_ext['module']
+            self.toggle_server_extension(module)

+def _get_server_extension_metadata(package):
 +    """Load server extension metadata from a package's magic-named path.
 +
 +    Returns a tuple of (
 +        the package as loaded
 +        a list of server extension specs: [
 +            {
 +                "module": "mockextension"
 +            }
 +        ]
 +    )
 +
 +    Parameters
 +    ----------
 +
 +    package : str
 +        Importable Python package (no dotted-notation!) exposing the
 +        magic-named `_jupyter_server_extension_paths` function
 +    """
 +    m = __import__(package)
 +    if not hasattr(m, '_jupyter_server_extension_paths'):
 +        raise KeyError('The Python package {} does not include any valid server extensions'.format(package))
 +    return m, m._jupyter_server_extension_paths()