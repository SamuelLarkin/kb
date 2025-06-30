# HuggingFace

## preprocess_logits_for_metrics

preprocess*logits_for_metrics (`Callable[[torch.Tensor, torch.Tensor], torch.Tensor]`, \_optional*): W [1/7]
A function that preprocess the logits right before caching them at each evaluation step.
Must take two tensors, the logits and the labels, and return the logits once processed as desired.
The modifications made by this function will be reflected in the predictions received by `compute_metrics`.

Note that the labels (second parameter) will be `None` if the dataset does not have them.
