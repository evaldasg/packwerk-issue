# packwerk-issue

#### Install npm package:

```shell
yarn add yaml-js
```

#### packwerk configured to exclude `node_modules`
```yaml
exclude:
- "{bin,node_modules,script,tmp,vendor}/**/*"
```

#### packwerk validation fails:
```shell
➜  packwerk-issue git:(main) bin/packwerk validate
📦 Packwerk is running validation...

Validation failed ❗
Unknown keys in /packwerk-issue/node_modules/yaml-js/src/package.yml: ["name", "version", "description", "main", "repository", "devDependencies", "license"]
If you think a key should be included in your package.yml, please open an issue in https://github.com/Shopify/packwerk
```

Suspected code: https://github.com/Shopify/packwerk/blob/main/lib/packwerk/package_set.rb#L37
```ruby
def package_paths(root_path, package_pathspec)
  bundle_path_match = Bundler.bundle_path.join("**").to_s

  glob_patterns = Array(package_pathspec).map do |pathspec|
    File.join(root_path, pathspec, PACKAGE_CONFIG_FILENAME)
  end

  Dir.glob(glob_patterns)
    .map { |path| Pathname.new(path).cleanpath }
    .reject { |path| path.realpath.fnmatch(bundle_path_match) }
end
```
This rejects only `gems` directory but does not take `@configuration.exclude` into account.
