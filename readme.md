###  FIX 1
file:./src/common/clib-package.c


ln 751
```
#ifdef HAVE_PTHREADS
      init_curl_share();
      _debug("GET %s", json_url);

     
      if (!res)
        res = http_get_shared(json_url, clib_package_curl_share);

#else
```


### FIX 2

```
#ifdef HAVE_PTHREADS
static void *fetch_package_file_thread(void *arg)
{

  fetch_package_file_thread_data_t *data = arg;
  int status = 0;
  int rc =
      fetch_package_file_work(data->pkg, data->dir, data->file, data->verbose);

  status = rc;
  (void)data->pkg->refs--;

  pthread_exit((void *)status);

  return (void *)rc;
}
#endif
```

### FIX 3

```
void clib_package_cleanup()
{
  if (0 != visited_packages)
  {
    hash_each(visited_packages, {
      free((void *)key);
      (void)val;
    });

    hash_free(visited_packages);
    visited_packages = 0;
  }
  // FIX
curl_share_cleanup(clib_package_curl_share);
#ifdef HAVE_PTHREADS
  pthread_exit(NULL);
#endif
}
```

### FIX 4

```
int clib_package_install(clib_package_t *pkg, const char *dir, int verbose){
...

    // hash_set(visited_packages, strdup(pkg->name), "t");

    if (!hash_has(visited_packages, pkg->name))
    {
      hash_set(visited_packages, strdup(pkg->name), "t");
    }

...
}
```