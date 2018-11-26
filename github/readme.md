# Github

## Github Markdown

### Use diff syntax in markdown

Use it to emphasize changes in question.

<pre>
```diff
- test: /\.(ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
+ test: /\.(ttf|eot|svg|gif)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
    include: SRC,
    use: [{
      loader: 'file-loader'
    }]
```
</pre>

To get something like:
```diff
- test: /\.(ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
+ test: /\.(ttf|eot|svg|gif)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
            include: SRC,
            use: [{
                loader: 'file-loader'
            }]
```
