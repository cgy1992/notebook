# Fragment newInstance 的实现

```
class NewsFragmet : Fragment() {

    companion object {
        fun newInstance(): NewsFragmet {
            val fragment = NewsFragmet()
            return fragment
        }
    }
```

