# Cyber Apocalypse 2023

## Mysterious Learnings

> As Pandora set out on her quest to find the ancient alien relic, she knew that the journey would be treacherous. The desert was vast and unforgiving, and the harsh conditions would put her cyborg body to the test. Pandora started by collecting data about the temperature and humidity levels in the desert. She used a scatter plot in an Orange Workspace file to visualize this data and identified the areas where the temperature was highest and the humidity was lowest. Using this information, she reconfigured her sensors to better withstand the extreme heat and conserve water. But, a second look at the data revealed something otherwordly, it seems that the relic's presence beneath the surface has scarred the land in a very peculiar way, can you see it?
>
>  Readme Author: [Fra-SM](https://github.com/Fra-SM)
>
> [`ml_mysterious_learnings.zip`](ml_mysterious_learnings.zip)

In this case we have a .h5 file, which is the file format used in Tensorflow to save and load keras models. With a bit of research, we can discover some of the main commands used to carve out information about the model:

```
model.summary
model.getConfig()
```
We can then retrieve the three base64 encoded pieces of our flag by running the following script using, for example, a Google Colab Notebook:
```
from tensorflow import keras

model = keras.models.load_model("alien.h5")
 
print(model.summary)
print(model.getConfig())
```
The final string put together is: SFRCe24wdF9zb19oNHJkX3RvX3VuZDNyc3Q0bmR9

## Flag
HTB{n0t_so_h4rd_to_und3rst4nd}